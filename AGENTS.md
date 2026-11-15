# AGENTS.md — Banners & Wars (Minecraft 1.20.1 Forge)

> **Propósito deste arquivo:** Este documento define o contexto arquitetural, padrões de engenharia, restrições de hardware e regras de fluxo de trabalho para agentes de IA compartilhados e desenvolvedores (Tech Lead + Estagiários) atuando neste repositório.

---

## 1. Visão Geral do Projeto
* **Nome do Mod Core:** `bannersandwars` (Modpack: *Banners & Wars*)
* **Plataforma:** Minecraft Java Edition **1.20.1** | Modloader: **Minecraft Forge (47.3.x)** | Java: **17 (Eclipse Temurin)**
* **Objetivo:** Transformar vilas em civilizações medievais autônomas com magia, economia baseada em armazéns, exploração, diplomacia e guerras em escala comandadas em primeira pessoa por meio de **Chifres de Cabra**.
* **Restrições Críticas de Ambiente:**
  1. Máquinas de desenvolvimento têm **16 GB de RAM** e **não possuem permissão de administrador** (tudo roda portátil via Gradle/IntelliJ/Prism).
  2. O servidor alvo é gratuito (**Oracle Cloud Always Free ARM — 4 OCPUs / 24 GB RAM**) e deve manter **20 TPS estáveis** com múltiplos jogadores e centenas de villagers ativos.

---

## 2. Regras Invioláveis de Arquitetura (Guardrails)

Qualquer código gerado ou revisado por agentes neste repositório **DEVE** obedecer às seguintes regras:

1. **NUNCA bloquear a Main Server Thread (`ServerTickEvent`):**
   * Inferência de Rede Neural (ONNX), consultas a banco de dados (SQLite/Nitrite) e cálculos de planejamento de longo prazo (`GOAP / Utility`) **DEVEM** rodar em `ExecutorService` assíncrono dedicado.
   * A *Main Thread* apenas lê `ActionIntent` imutáveis já calculadas e executa movimentação/animação nativa do Minecraft.
2. **NUNCA rodar IA dos Villagers no Cliente do Jogador:**
   * O servidor detém 100% da autoridade lógica e de IA.
   * O **único** processamento pesado permitido no cliente é a transcrição local de voz (`Vosk STT`), que envia apenas uma `String` sanitizada via pacote de rede (`SimpleChannel`) para o servidor.
3. **Simulação Macro para Vilas Distantes (Unloaded Chunks):**
   * Vilas fora da distância de renderização dos jogadores **não instanciam entidades `Entity`**. Elas operam como registros em memória/banco atualizados a cada 100 ticks (5 segundos) e são materializadas no mundo físico apenas quando um jogador se aproxima.
4. **Pathfinding em Esquadrão (Boids / Squads):**
   * Durante combates e marchas militares, **apenas o `VillagerGeneral` calcula pathfinding A***. Os soldados subordinados calculam apenas vetores de deslocamento relativo à formação do general.
5. **Reutilização Antes de Criação:**
   * Não reescreva sistemas de combate corpo a corpo, renderização de entidades animadas ou sistema base de mana/feitiços. Utilize os hooks dos mods parceiros documentados em `/docs/MODPACK_MANIFEST.md` (`Better Combat`, `Iron's Spells 'n Spellbooks`, `GeckoLib`, `AstikorCarts`).
6. **Segurança de Escrita em Banco de Dados:**
   * Toda escrita no **SQLite** (dados relacionais) e **Nitrite DB** (documentos NoSQL) deve ocorrer via fila assíncrona (`Write-Behind Queue`) e ser descarregada (flush) obrigatoriamente no evento `ServerStoppingEvent`.

---

## 3. Anatomia Cognitiva do Villager (5 Módulos Obrigatórios)

Todo agente inteligente (`KingdomVillagerEntity`) possui um `VillagerBrain` composto por cinco módulos desacoplados:

1. **`ObjectiveModule` (Objetivo):** Ação atual + justificativa causal legível (ex.: `"Coletar madeira pois o construtor Junior abriu requisição no Armazém"`).
2. **`RelationsModule` (Relações):** Grafo de afinidade com outros villagers e jogadores + histórico recente de diálogos/fofocas (persistido em NoSQL).
3. **`RoutineModule` (Rotina):** Agenda diária dinâmica (horário de trabalho, refeições no Restaurante, oração no Templo, sono).
4. **`PersonalityModule` (Personalidade):** Vetor numérico `[Coragem, Ganância, Sociabilidade, Devoção, Diligência]` que altera os pesos da Rede Neural e o tom dos diálogos.
5. **`ContextModule` (Outros):** *Heatmap* espacial de perigo/recursos, necessidades fisiológicas (fome/energia), moral/lealdade ao reino e XP individual da profissão.

---

## 4. Convenções de Código e Estrutura de Pacotes

```text
src/main/java/com/mojang/bannersandwars/
├── core/           # Registro de blocos, itens, entidades, redes e eventos Forge
├── ai/
│   ├── brain/      # Os 5 módulos cognitivos e ActionIntents
│   ├── onnx/       # Wrapper do ONNX Runtime para inferência Java
│   └── voice/      # Integração cliente Vosk STT e pacotes de chat
├── kingdom/        # Loteamento por chunks (15/40/100), Banners, Armazém e Simulação Macro
├── profession/     # Implementação modular de cada profissão (1 pacote por profissão)
├── war/            # Generais, Esquadrões (Boids), Moral, Chifres de Cabra e Cerco
├── persistence/    # Repositórios SQLite (relacional) e Nitrite (NoSQL)
└── client/         # UIs (Criação de Banner, Mapa-Múndi, Chat/Voz) e renderers GeckoLib

python_ai/          # Scripts Python (PyTorch) para treino das redes neurais e exportação .onnx
```

* **Nomenclatura:** Classes em `PascalCase`, métodos/variáveis em `camelCase`, constantes em `UPPER_SNAKE_CASE`.
* **Idioma do Código:** Classes, métodos, variáveis e commits em **Inglês**. Strings de interface (arquivos `pt_br.json`) e documentação em **Português (BR)**.

---

## 5. Fluxo Git e Definition of Done (DoD)

* **Branches:**
  * `main`: Versão estável testada no servidor.
  * `develop`: Branch de integração das sprints.
  * `feat/<profissao-ou-sistema>`: Uma branch isolada por profissão ou mecânica (ex.: `feat/prof-lumberjack`, `feat/goat-horns`).
* **Definition of Done (DoD) para qualquer Pull Request:**
  1. Compila sem erros via `./gradlew build`.
  2. Nenhuma chamada de I/O ou inferência pesada na *Main Thread* (verificado via comando `/spark profiler` em teste local).
  3. Persistência testada: salvar mundo, fechar jogo, reabrir e confirmar integridade no SQLite/Nitrite.
  4. Revisado e aprovado pelo Tech Lead.
