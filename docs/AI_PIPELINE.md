# AI_PIPELINE.md — Treinamento em Python, ONNX e Voz Offline

## 1. Objetivo Didático e Técnico
Os estagiários não usarão APIs pagas de nuvem nem LLMs pesados que estouram a memória RAM. O projeto implementa **Redes Neurais reais treinadas em Python (`PyTorch`)** e exportadas para o formato aberto **ONNX**, executadas nativamente em Java via `com.microsoft.onnxruntime:onnxruntime`.

---

## 2. Arquitetura da Rede Neural de Decisão (`VillagerUtilityNet`)

### 2.1. Tensor de Entrada (16 Features Normalizadas `[0.0, 1.0]`)
1. `time_of_day` (0.0 a 1.0 representando as 24h do Minecraft)
2. `hunger_level` (0.0 = saciado, 1.0 = faminto)
3. `energy_level` (0.0 = descansado, 1.0 = exausto)
4. `personality_courage`
5. `personality_greed`
6. `personality_social`
7. `personality_devotion`
8. `personality_diligence`
9. `warehouse_wood_ratio` (estoque atual / meta do armazém)
10. `warehouse_stone_ratio`
11. `warehouse_food_ratio`
12. `pending_build_requests` (0.0 a 1.0)
13. `village_threat_level` (proximidade de inimigos ou guerra ativa)
14. `kingdom_morale`
15. `grief_level` (luto ativo por funeral recente)
16. `profession_xp_norm`

### 2.2. Topologia da Rede (MLP Leve)
* **Input Layer:** `16` neurônios
* **Hidden Layer 1:** `32` neurônios + `ReLU`
* **Hidden Layer 2:** `16` neurônios + `ReLU`
* **Output Layer:** `7` neurônios + `Softmax` (probabilidade de escolha da meta macro):
  1. `WORK_PROFESSION` (Executar tarefa da profissão)
  2. `EAT_AT_RESTAURANT` (Ir comer e socializar)
  3. `PRAY_AT_TEMPLE` (Ir rezar e recuperar moral)
  4. `SLEEP_AT_HOME` (Ir dormir)
  5. `SOCIALIZE_PLAZA` (Conversar/fofocar na praça)
  6. `FLEE_TO_SAFEZONE` (Fugir de ameaça)
  7. `RALLY_FOR_WAR` (Apresentar-se ao General do esquadrão)

---

## 3. Fluxo de Trabalho Python -> Java

```bash
# 1. Ativar ambiente virtual Python
cd python_ai
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install torch onnx numpy pandas

# 2. Gerar dataset sintético baseado nas regras de Rotina e Personalidade
python scripts/generate_dataset.py --samples 50000 --out data/villager_decisions.csv

# 3. Treinar o modelo PyTorch e exportar para ONNX
python scripts/train_and_export.py --epochs 40 --out ../src/main/resources/assets/bannersandwars/models/villager_brain.onnx
```

* **Performance esperada:** O arquivo `.onnx` final pesa **menos de 25 KB** e executa **1.000 inferências em ~12 milissegundos** na CPU do servidor.

---

## 4. Pipeline de Reconhecimento de Voz no Cliente (`Vosk STT`)

1. No lado do cliente (`Dist.CLIENT`), carregamos o modelo compacto **`vosk-model-small-pt-0.3`** (~45 MB em RAM) em uma *thread* dedicada na inicialização do jogo.
2. Quando o jogador pressiona a tecla configurada (`V` por padrão) olhando para um villager num raio de 6 blocos:
   * O microfone captura áudio PCM 16kHz mono via `javax.sound.sampled.TargetDataLine`.
   * Ao soltar a tecla, o `Recognizer` do Vosk finaliza a string (ex.: *"Ei Junior, do que você precisa para terminar essa casa?"*).
3. O cliente envia `C2SVillagerChatPacket` ao servidor.
4. O servidor cruza palavras-chave/intenção com o `ObjectiveModule` e `RelationsModule` do villager e devolve a resposta contextualizada no chat.
