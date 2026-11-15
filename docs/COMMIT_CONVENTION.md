# COMMIT_CONVENTION.md — Guia de Commits e Versionamento

Este documento define o padrão de mensagens de commit adotado no projeto **Banners & Wars**, baseado na especificação internacional **[Conventional Commits v1.0.0](https://www.conventionalcommits.org/)**.

Todos os membros da equipe (Tech Lead, Desenvolvedores e Agentes de IA) devem seguir estritamente estas diretrizes.

---

## 1. Estrutura Padrão da Mensagem

```text
<tipo>[escopo opcional]: <descrição curta e imperativa>

[corpo opcional explicando o contexto e o "porquê"]

[rodapé(s) opcional(is) - ex: referências a issues, breaking changes]
```

### Regras Gerais:
1. **Idioma:** Mensagens de commit devem ser escritas em **Inglês** (conforme definido na convenção do `AGENTS.md`).
2. **Tempo Verbal:** Use o imperativo no presente (ex.: `add`, `fix`, `refactor`, e não `added`, `fixing`, `fixes`).
3. **Sem Ponto Final:** A linha de título não termina com ponto final (`.`).
4. **Limite de Caracteres:** Máximo de **72 caracteres** na primeira linha.

---

## 2. Tipos de Commit Permitidos

| Tipo | Descrição | Exemplo |
| :--- | :--- | :--- |
| **`feat`** | Nova funcionalidade ou mecânica adicionada ao jogo | `feat(brain): implement routine module daily schedule` |
| **`fix`** | Correção de bug ou comportamento inesperado | `fix(warehouse): prevent item duplication on inventory reservation` |
| **`docs`** | Alterações exclusivas na documentação | `docs: add initial project architecture and GDD` |
| **`refactor`** | Refatoração de código sem alterar regra de negócio ou adicionar feature | `refactor(network): optimize packet serialization for world map sync` |
| **`perf`** | Melhoria de desempenho (TPS, CPU ou consumo de memória RAM) | `perf(onnx): batch neural inference calls in async worker pool` |
| **`test`** | Adição ou refatoração de testes automatizados | `test(sqlite): add unit tests for kingdom territory claim queries` |
| **`build`** | Alterações no sistema de build, Gradle ou dependências externas | `build: bump forge version to 47.3.7 and add sqlite-jdbc` |
| **`ci`** | Alterações em scripts de integração contínua (GitHub Actions) | `ci: add workflow to check gradlew build on PRs` |
| **`style`** | Formatação de código, pontuação ou imports sem impacto em lógica | `style(core): format imports according to checkstyle rules` |
| **`chore`** | Tarefas de manutenção rotineira, `.gitignore`, configs de IDE | `chore: update .gitignore to exclude runClient cache and sqlite test db` |

---

## 3. Escopos Sugeridos (`scope`)

O escopo ajuda a identificar qual subsistema do modpack foi afetado:

* `core` — Registros de blocos, itens, eventos Forge e modloader.
* `brain` — Módulos cognitivos dos villagers (`Objective`, `Relations`, `Routine`, `Personality`, `Context`).
* `onnx` — Integração com runtime ONNX e inferência de redes neurais.
* `voice` — Vosk STT, captura de microfone e processamento de áudio cliente.
* `kingdom` — Reivindicação de chunks, estandartes e simulação macro.
* `profession` — Implementação das profissões (`lumberjack`, `builder`, `farmer`, etc.).
* `war` — Generais, formações de esquadrões (Boids), moral e Chifres de Cabra.
* `persistence` — Repositórios SQLite (`kingdoms.db`) e Nitrite NoSQL (`memories.nitrite`).
* `network` — Pacotes de rede `SimpleChannel` (C2S e S2C).
* `client` — Telas (GUI), renderers GeckoLib e HUD.
* `python` — Scripts de treinamento em PyTorch, geração de datasets sintéticos e exportação de modelos.

---

## 4. Exemplos Práticos

### Nova Mecânica (Feature)
```text
feat(war): add horn blowing listeners and squad command dispatch

Implement Bronze and Silver goat horns triggering charge and regroup
orders to nearby general entities within a 64-block radius.
```

### Correção de Bug Crítico
```text
fix(concurrency): avoid running onnx inference on server main thread

Offload neural evaluation to AsyncBrainExecutor to prevent dropping ticks
during village routine recalculation.
```

### Breaking Change
Quando houver quebra de compatibilidade em schemas de banco ou contratos de rede, adicione `!` antes do `:` ou declare `BREAKING CHANGE:` no rodapé:

```text
feat(persistence)!: migrate kingdom territory schema to support multiple dimensions

BREAKING CHANGE: The claimed_chunks table now requires dimension identifier
as part of its compound primary key.
```

---

## 5. Fluxo de Branches e Pull Requests

1. **Nomeação de Branches:**
   * `feat/<modulo-ou-sistema>` (ex.: `feat/prof-lumberjack`, `feat/goat-horns`)
   * `fix/<descricao-do-bug>` (ex.: `fix/main-thread-block`)
   * `docs/<tema>` (ex.: `docs/add-agents-guide`)
2. **Squash & Merge:**
   * Ao finalizar o PR para `develop`, prefira realizar *Squash and Merge* mantendo um único commit limpo e convencional no histórico principal.
