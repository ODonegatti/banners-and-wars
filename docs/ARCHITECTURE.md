# ARCHITECTURE.md — Engenharia de Performance & Dados

## 1. Visão Geral da Arquitetura em 3 Camadas

Para garantir **20 TPS** em servidores gratuitos (Oracle Cloud ARM) e clientes com **16 GB de RAM**, o processamento é dividido em três camadas isoladas por *threads*:

```text
+-----------------------------------------------------------------------+
|                        CLIENTE (16 GB RAM Cap)                        |
|  [Render: Embeddium + GeckoLib]   [Voz Offline: Vosk STT (~50 MB)]    |
+-----------------------------------------+-----------------------------+
                                          | Netty Packet (String Texto)
                                          v
+-----------------------------------------------------------------------+
|                     SERVIDOR (Minecraft 1.20.1 Forge)                 |
|                                                                       |
|  [CAMADA 1: Main Server Thread (20 TPS)]                              |
|   -> Executa ActionIntents prontas (andar, quebrar bloco, bater)      |
|   -> Esquadrões Boids (Apenas General usa A*; soldados seguem vetor)  |
|                                                                       |
|  [CAMADA 2: Cognitive Thread Pool (Async - 4 Workers)]                |
|   -> Inferência ONNX Runtime (< 0.1ms por villager)                   |
|   -> Atualização de Objetivo, Rotina, Relações e Moral                |
|                                                                       |
|  [CAMADA 3: Macro Kingdom Simulator (Async - 1 Worker / 5s)]          |
|   -> Simula vilas em unloaded chunks sem spawnar entidades            |
|                                                                       |
|  [CAMADA DE PERSISTÊNCIA (Async Write-Behind)]                        |
|   -> SQLite (Relacional)           -> Nitrite DB (NoSQL Documentos)   |
+-----------------------------------------------------------------------+
```

---

## 2. Persistência Híbrida (SQLite + Nitrite DB)

Os arquivos do banco são salvos dentro da pasta do mundo: `<world_folder>/realmsandhorns_data/`.

### 2.1. Banco Relacional (`kingdoms.db` via SQLite JDBC)
Armazena integridade referencial, inventários, posse territorial e genealogia.

```sql
CREATE TABLE kingdoms (
    kingdom_id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    owner_uuid TEXT, -- NULL se for vila autônoma sem player
    banner_nbt TEXT NOT NULL,
    tier TEXT CHECK(tier IN ('VILLAGE', 'CAPITAL', 'KINGDOM')) DEFAULT 'VILLAGE',
    morale REAL DEFAULT 75.0,
    created_at INTEGER NOT NULL
);

CREATE TABLE claimed_chunks (
    chunk_x INTEGER NOT NULL,
    chunk_z INTEGER NOT NULL,
    dimension TEXT NOT NULL,
    kingdom_id TEXT NOT NULL,
    PRIMARY KEY (chunk_x, chunk_z, dimension),
    FOREIGN KEY (kingdom_id) REFERENCES kingdoms(kingdom_id) ON DELETE CASCADE
);

CREATE TABLE villagers (
    villager_uuid TEXT PRIMARY KEY,
    kingdom_id TEXT NOT NULL,
    full_name TEXT NOT NULL,
    profession TEXT NOT NULL,
    profession_xp INTEGER DEFAULT 0,
    is_alive INTEGER DEFAULT 1,
    father_uuid TEXT,
    mother_uuid TEXT,
    spouse_uuid TEXT,
    personality_vector TEXT NOT NULL, -- JSON array: [courage, greed, social, devotion, diligence]
    FOREIGN KEY (kingdom_id) REFERENCES kingdoms(kingdom_id)
);

CREATE TABLE warehouse_inventory (
    kingdom_id TEXT NOT NULL,
    item_id TEXT NOT NULL,
    quantity INTEGER DEFAULT 0,
    reserved_quantity INTEGER DEFAULT 0, -- Reservado por construtores/ferreiros
    PRIMARY KEY (kingdom_id, item_id),
    FOREIGN KEY (kingdom_id) REFERENCES kingdoms(kingdom_id)
);

CREATE TABLE funerals (
    funeral_id INTEGER PRIMARY KEY AUTOINCREMENT,
    kingdom_id TEXT NOT NULL,
    is_mass_war_funeral INTEGER DEFAULT 0,
    casualties_count INTEGER NOT NULL,
    occurred_at INTEGER NOT NULL
);
```

### 2.2. Banco NoSQL (`memories.nitrite` via Nitrite DB)
Armazena documentos flexíveis e de leitura rápida por UUID do villager, sem necessidade de `JOINs` pesados durante o jogo.

```json
{
  "villager_uuid": "a81bc81b-dead-4e5d-abff-90865d1e13b1",
  "current_objective": {
    "action": "GATHER_WOOD",
    "reason": "O construtor Junior requisitou 32 tábuas de carvalho no Armazém",
    "priority_score": 0.87,
    "timestamp": 1794859200
  },
  "relations": [
    {
      "target_uuid": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "target_name": "Junior",
      "affinity": 82,
      "last_interactions": [
        "Almoçaram juntos no Restaurante e falaram sobre a expansão da muralha.",
        "Junior agradeceu pela entrega rápida de madeira."
      ]
    }
  ],
  "spatial_heatmap": {
    "safe_resources": [
      {"x": 120, "y": 68, "z": -340, "type": "OAK_FOREST", "score": 0.9}
    ],
    "danger_zones": [
      {"x": -45, "y": 64, "z": 210, "reason": "Batalha recente - 4 aliados mortos", "danger_level": 0.95}
    ]
  }
}
```

---

## 3. Protocolo de Rede Customizado (`RealmsNetwork`)

Todos os pacotes usam `SimpleChannel` do Forge com serialização compacta:

| Pacote | Direção | Payload | Frequência |
| :--- | :--- | :--- | :--- |
| `C2SBannerCreatePacket` | Client -> Server | NBT do Estandarte + Nome do Reino | 1x no primeiro login |
| `C2SVillagerChatPacket` | Client -> Server | `UUID villagerId`, `String message` (texto digitado ou transcrito pelo Vosk) | Sob demanda |
| `S2CVillagerReplyPacket` | Server -> Client | `UUID villagerId`, `String reply`, `String currentObjective` | Sob demanda |
| `S2CHornCommandAlertPacket` | Server -> Client | `Enum HornType`, `int affectedSquads` | Ao soprar Chifre |
| `S2CLeaderDefeatedPacket` | Server -> Client | `String generalName`, `String kingdomName`, `boolean isAlly` | Na morte de um General |
| `S2CWorldMapSyncPacket` | Server -> Client | Lista comprimida (GZIP) de vilas descobertas e fronteiras | Ao abrir Mesa de Mapa |
