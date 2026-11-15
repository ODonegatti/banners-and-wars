# Banners & Wars

![Minecraft](https://img.shields.io/badge/Minecraft-1.20.1-62B47A?style=for-the-badge&logo=minecraft&logoColor=white)
![Modloader](https://img.shields.io/badge/Forge-47.3.x-1E293B?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17%20(Temurin)-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![AI](https://img.shields.io/badge/AI-PyTorch%20%E2%86%92%20ONNX-EE4C2C?style=for-the-badge&logo=onnx&logoColor=white)
![Database](https://img.shields.io/badge/DB-SQLite%20%2B%20Nitrite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)

> **Civilizações medievais autônomas, magia arcana, diplomacia e guerras em escala comandadas em primeira pessoa — otimizado para rodar a 20 TPS em servidores gratuitos e máquinas de 16 GB de RAM.**

---

## Visão Geral

**Banners & Wars** (`mod_id: bannersandwars`) é um modpack e motor de simulação para **Minecraft Java Edition 1.20.1 (Forge)** que transforma aldeias estáticas em reinos vivos e autônomos. 

Cada villager possui identidade própria, genealogia, memória episódica e um cérebro movido por **Redes Neurais leves (ONNX)** combinadas com reconhecimento de voz offline (**Vosk STT**). Mesmo sem intervenção de jogadores, vilas evoluem lentamente sob liderança política local; quando reivindicadas pelo estandarte de um jogador, expandem-se em cidades e reinos capazes de travar guerras táticas comandadas por **Chifres de Cabra**.

---

## Pilares de Gameplay

### Território, Estandartes e Fog of War Político
* **Estandarte de Linhagem:** No primeiro login, o jogador desenha seu próprio estandarte para reivindicar vilas neutras ou fundar novos assentamentos.
* **Escala Territorial (Chunks 16×16):**
  * **Vila Pequena:** `15 chunks`
  * **Capital:** `40 chunks`
  * **Reino:** `100 chunks`
* **Descoberta de Mundo:** Vilas sem jogador desconhecem umas às outras até serem descobertas por jogadores ou **Villagers Exploradores**, que mapeiam o terreno e alimentam o **Mapa-Múndi** da Guilda dos Cartógrafos.

### Cérebro Cognitivo em 5 Módulos + Voz Offline
Cada villager opera sobre cinco pilares desacoplados:
1. **Objetivo (`Objective`):** Ação atual e motivo causal explícito (*ex.: "Cortar carvalho pois o construtor Junior requisitou 32 tábuas no Armazém"*).
2. **Relações (`Relations`):** Grafo de afinidade (`-100` a `+100`) e histórico de conversas/fofocas com outros villagers e jogadores.
3. **Rotina (`Routine`):** Horários dinâmicos de trabalho, refeições no Restaurante, oração no Templo e sono.
4. **Personalidade (`Personality`):** Vetor `[Coragem, Ganância, Sociabilidade, Devoção, Diligência]` que altera decisões neurais e diálogos.
5. **Contexto (`Context`):** *Heatmaps* espaciais de recursos/perigo, fome, moral e XP individual da profissão.
* **Chat & Voz (Push-to-Talk):** O cliente transcreve a fala do jogador localmente via **Vosk (~45 MB RAM)** e envia apenas texto leve ao servidor.

### Economia de Armazém e Cadeia de Profissões
Todas as profissões são integradas ao **Armazém Central** e construídas modularmente:
* **Infraestrutura:** Lenhador, Minerador, Construtor Civil (*Cabana -> Casa -> Mansão -> Castelo*) e Construtor Viário (terraplanagem e estradas).
* **Alimentação & Social:** Fazendeiro, Pecuarista e Cozinheiro (opera o **Restaurante**, principal hub de socialização).
* **Política & Exploração:** Político (Prefeito/Ancião) e Explorador/Cartógrafo (mensageiro e criador de mapas).
* **Forja, Fé & Arcano:** Ferreiro, Encantador e Clérigo (atua no Templo e cura tropas em combate).

### Ciclo Vital: Casamentos, Nascimentos e Funerais
* Alta afinidade entre villagers adultos + moradia disponível disparam eventos públicos de **Casamento**, gerando novos habitantes que herdam traços dos pais.
* **Morte Permanente:** Todo villager abatido recebe um **Funeral** no templo/cemitério (ou um **Funeral Coletivo** após guerras), gerando luto e impactando a moral e produtividade dos sobreviventes.

### Guerra Tática, Generais e Chifres de Cabra
* **Unidades Militares:** Infantaria, Arqueiros, Cavalaria (montados), Magos ofensivos e Clérigos de suporte, organizados em **Esquadrões** liderados por **Villagers Generais**.
* **Comandos em 1ª Pessoa:** Enquanto luta na linha de frente, o jogador sopra diferentes **Chifres de Cabra** para ordenar os Generais próximos (*Avançar, Recuar, Cercar/Flanquear, Manter Posição*).
* **Queda de Líder:** Derrotar um General inimigo dispara alerta sonoro e visual (`[Líder Derrotado]`), derrubando a moral adversária.
* **Cerco Anti-Frustração:** Apenas **Muralhas e Portões** podem ser destruídos em guerra; edificações civis permanecem intactas e a vitória ocorre pela captura do **Estandarte Central / Trono**.

---

## Arquitetura & Performance (20 TPS / 16 GB RAM)

Para suportar dezenas de jogadores e reinos sem sobrecarregar servidores gratuitos ou PCs de desenvolvimento com 16 GB de RAM:

* **Execução Assíncrona em 3 Camadas:**
  1. *Main Server Thread:* Apenas movimentação física e execução de `ActionIntents` prontas.
  2. *Cognitive Thread Pool:* Inferência das redes neurais **ONNX** (`< 0.1 ms` por agente) fora do loop principal.
  3. *Macro Kingdom Simulator:* Vilas em *unloaded chunks* evoluem como simulação de dados a cada 5 segundos, sem instanciar entidades.
* **Boids / Esquadrões:** Em batalhas, apenas o `VillagerGeneral` calcula pathfinding A*; soldados seguem vetores relativos de formação.
* **Persistência Híbrida Embarcada:**
  * **SQLite (`kingdoms.db`):** Chunks, estoques do armazém, genealogia e registros de funerais.
  * **Nitrite DB (`memories.nitrite`):** Documentos NoSQL para memórias episódicas, diálogos e *heatmaps* espaciais.
* **Stack Reutilizada no Modpack:** `Embeddium`, `FerriteCore`, `ModernFix`, `Radium`, `Spark`, `GeckoLib`, `Epic Knights`, `Better Combat`, `Iron's Spells 'n Spellbooks` e `AstikorCarts`.

---

## Estrutura do Repositório

```text
banners-and-wars/
├── AGENTS.md                  # Guardrails e instruções para agentes de IA compartilhados
├── docs/
│   ├── GDD.md                 # Game Design Document completo
│   ├── ARCHITECTURE.md        # Arquitetura multi-thread, schemas SQLite/Nitrite e rede
│   ├── AI_PIPELINE.md         # Pipeline Python (PyTorch -> ONNX) e Voz (Vosk)
│   ├── CRONOGRAMA.md          # Roadmap de 16 semanas (8 Sprints) e divisão de tarefas
│   ├── MODPACK_MANIFEST.md    # Mods 1.20.1 reutilizados e pontos de integração
│   └── SETUP_DEV.md           # Setup portátil sem permissão admin + Oracle Cloud Free
├── python_ai/                 # Scripts Python (PyTorch) para treino das redes neurais e exportação .onnx
└── src/main/java/com/mojang/bannersandwars/
    ├── core/                  # Registro de blocos, itens, entidades, redes e eventos Forge
    ├── ai/                    # Cérebro cognitivo (5 módulos), runtime ONNX e cliente Vosk
    ├── kingdom/               # Chunks (15/40/100), Estandartes, Armazém e Simulação Macro
    ├── profession/            # Implementação modular das profissões (1 pacote por profissão)
    ├── war/                   # Generais, Esquadrões (Boids), Moral, Chifres e Cerco
    ├── persistence/           # Repositórios assíncronos SQLite e Nitrite DB
    └── client/                # Telas (Criação de Banner, Mapa-Múndi) e renderers GeckoLib
```

---

## Quickstart (Ambiente Portátil Sem Permissão Admin)

O ambiente foi projetado para rodar integralmente em diretório de usuário (sem instaladores `.exe`/`.msi` e sem necessidade do Launcher oficial da Mojang):

1. **Requisitos Portáteis:**
   * **Eclipse Temurin JDK 17** (`.zip` extraído localmente)
   * **IntelliJ IDEA Community** (`.zip` portátil)
   * **Python 3.11+** (para pipeline de treino da IA)
2. **Clonar e Compilar:**
   ```bash
   git clone https://github.com/<sua-org>/banners-and-wars.git
   cd banners-and-wars
   ./gradlew build
   ```
3. **Treinar e Exportar a Rede Neural Base (`.onnx`):**
   ```bash
   cd python_ai
   python -m venv .venv
   .venv\Scripts\activate
   pip install torch onnx numpy pandas
   python scripts/generate_dataset.py --samples 50000 --out data/villager_decisions.csv
   python scripts/train_and_export.py --epochs 40 --out ../src/main/resources/assets/bannersandwars/models/villager_brain.onnx
   ```
4. **Executar Cliente de Teste (Autenticação Microsoft OAuth):**
   ```bash
   ./gradlew runClient
   ```
   > *Para testar o modpack completo fora da IDE, utilize o **Prism Launcher Portable** com alocação máxima de **6144 MB (6 GB)** de RAM.*

---

## Documentação Completa

* **[AGENTS.md](./AGENTS.md)** — Regras invioláveis de arquitetura e padrões para agentes de código.
* **[Game Design Document (GDD)](./docs/GDD.md)** — Detalhamento de mecânicas, profissões, guerra e sociedade.
* **[Arquitetura Técnica](./docs/ARCHITECTURE.md)** — Diagramas de *threads*, tabelas SQLite, documentos Nitrite e pacotes de rede.
* **[Pipeline de IA & Voz](./docs/AI_PIPELINE.md)** — Topologia da rede neural, treino em Python e configuração do Vosk STT.
* **[Cronograma & Sprints](./docs/CRONOGRAMA.md)** — Plano de 16 semanas e backlog detalhado da equipe.
* **[Manifesto do Modpack](./docs/MODPACK_MANIFEST.md)** — Lista de mods integrados e fronteiras de desenvolvimento.
* **[Guia de Setup & Servidor Gratuito](./docs/SETUP_DEV.md)** — Configuração portátil e deploy na Oracle Cloud Always Free.

---

## Fluxo de Contribuição (GitFlow)

* `main` — Código estável e validado em servidor multiplayer a 20 TPS.
* `develop` — Branch de integração contínua das Sprints.
* `feat/<sistema-ou-profissao>` — Cada nova profissão ou mecânica deve ser desenvolvida em branch isolada (ex.: `feat/prof-lumberjack`, `feat/goat-horns`) e validada com `/spark profiler` antes do *Pull Request*.
