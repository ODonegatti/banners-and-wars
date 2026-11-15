# CRONOGRAMA.md — Plano de Execução (16 Semanas / 8 Sprints)

## 1. Estrutura da Equipe e Alocação
* **Tech Lead (Senior Programmer):** Arquitetura *multi-thread*, integração com APIs de mods externos, revisão de PRs e otimização de TPS.
* **Estagiário A (Foco: IA, Dados e Sociedade):** Redes Neurais (Python/ONNX), SQLite + Nitrite DB, Voz (Vosk), Cadeia Alimentar, Casamentos e Funerais.
* **Estagiário B (Foco: Mundo, Construção e Guerra):** Chunks/Estandartes, Armazém, Construtores/Terraplanagem, Exploradores/Mapa-Múndi e Sistema de Guerra/Chifres.

---

## 2. Cronograma Macro por Sprints (2 Semanas cada)

| Sprint | Semanas | Foco Principal | Entregável Validado (Milestone) |
| :--- | :--- | :--- | :--- |
| **Sprint 1** | Sem 1–2 | Setup portátil, Bancos de Dados, Estandarte e 1ª Profissão | Player cria Estandarte, reivindica 15 chunks e **Lenhador** corta madeira e guarda no **Armazém**. |
| **Sprint 2** | Sem 3–4 | Rede Neural v1 (ONNX) + Construtores Civil e Viário | **Minerador** ativo; **Construtor** ergue Cabana -> Casa usando estoque do Armazém; estradas conectadas. |
| **Sprint 3** | Sem 5–6 | Cadeia Alimentar + Restaurante + Voz (Vosk) | **Fazendeiro, Pecuarista e Cozinheiro** operando; villagers almoçam no Restaurante e respondem chat/voz. |
| **Sprint 4** | Sem 7–8 | Ciclo Vital (Casamentos/Funerais) + Simulação Macro | Vilas evoluem em *unloaded chunks*; casamentos geram filhos; mortes geram funerais e luto no NoSQL. |
| **Sprint 5** | Sem 9–10 | Política Autônoma + Exploradores + Mapa-Múndi | Vilas desconhecidas isoladas; **Explorador** descobre vilas, entrega cartas e preenche **Mapa-Múndi**. |
| **Sprint 6** | Sem 11–12 | Forja, Encantamento, Templo e Clérigos | **Ferreiro** cria armas (*Epic Knights*), **Encantador** encanta e **Clérigo** reza/cura aliados. |
| **Sprint 7** | Sem 13–14 | Esquadrões (Boids), Generais e Chifres de Cabra | Exércitos marcham em formação; player comanda tropas em 1ª pessoa com **Chifres de Cabra**. |
| **Sprint 8** | Sem 15–16 | Cerco de Muralhas, Moral, Balanceamento e Deploy OCI | Destruição restrita a muralhas, captura de Trono, evento **Líder Derrotado** e teste de estresse 20 TPS. |

---

## 3. Backlog Detalhado — Sprint 1 (Kickoff Imediato)

### Tarefas do Tech Lead
* **[TL-01]** Criar repositório GitHub com regras de proteção (`main` / `develop`) e publicar `AGENTS.md` e `/docs`.
* **[TL-02]** Configurar projeto base Forge 1.20.1 MDK com dependências (`sqlite-jdbc`, `nitrite`, `onnxruntime`, `vosk`, `geckolib`).
* **[TL-03]** Implementar o `AsyncBrainExecutor` (pool de threads seguro) para os estagiários plugarem os agentes.

### Tarefas do Estagiário A (IA & Dados)
* **[EA-01]** Criar `DatabaseManager` inicializando `kingdoms.db` (SQLite) e `memories.nitrite` ao carregar o mundo.
* **[EA-02]** Implementar `VillagerBrain` com os 5 módulos base e comando `/villager inspect` para depurar Objetivo, Rotina e Personalidade no chat.
* **[EA-03]** Criar protótipo inicial em Python (`python_ai/`) gerando vetores de personalidade e exportando o primeiro `.onnx` dummy.

### Tarefas do Estagiário B (Mundo & Profissões)
* **[EB-01]** Criar tela `BannerCreatorScreen` exibida no primeiro login do jogador e salvar o NBT do estandarte no SQLite.
* **[EB-02]** Implementar bloco `VillageCenterBlock` (Pedestal) que reivindica raio de **15 chunks** e bloqueia construção de não-aliados.
* **[EB-03]** Implementar bloco `WarehouseBlock` (Armazém Central) + entidade/profissão **Lenhador** (busca árvore -> corta -> deposita no Armazém).
