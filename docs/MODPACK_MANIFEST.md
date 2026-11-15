# MODPACK_MANIFEST.md — Stack 1.20.1 Forge e Hooks de Integração

Para maximizar a produtividade dos estagiários e manter o consumo abaixo de **8 GB de RAM no cliente**, utilizamos os seguintes mods consolidados no modpack:

## 1. Performance e Otimização (Obrigatórios)
* **Embeddium** (Renderização rápida no cliente — substitui Sodium no Forge).
* **FerriteCore** (Redução drástica de uso de RAM por modelos de blocos e estados).
* **ModernFix** (Carregamento rápido do modpack e correção de vazamentos de memória).
* **Radium Reforged** (Otimização de física, IA vanilla e ticks no servidor — port do Lithium).
* **Spark** (Profiler de CPU/Memória para auditar se os agentes não estão derrubando os 20 TPS).

## 2. Conteúdo, Combate e Magia (APIs Reutilizadas)
* **GeckoLib (4.4.x):** Usado para renderizar e animar os modelos 3D customizados dos villagers (armaduras, vestes de mago, mochilas de explorador, animações de ataque e funeral) criados via IA + Blockbench.
* **Epic Knights: Shields, Armor and Weapons:** Fornece todo o arsenal medieval (espadas longas, alabardas, escudos, armaduras de placas) que o **Ferreiro** fabrica e equipa nos soldados.
* **Better Combat:** Melhora as animações e *hitboxes* de combate corpo a corpo tanto para o jogador quanto para os villagers soldados.
* **Iron's Spells 'n Spellbooks:** Fornece o sistema de mana, pergaminhos e feitiços usados pelos **Villagers Magos** (bolas de fogo, raios, escudos arcanos) e **Clérigos** (curas em área).
* **AstikorCarts:** Fornece carroças puxadas por cavalos para transporte de grandes cargas entre assentamentos e rotas comerciais.

## 3. Onde o Código dos Estagiários Entra (Fronteira Clara)
* **NÃO criar:** Novos minérios genéricos, novas espadas base, novo motor de renderização 3D ou novo sistema de feitiços do zero.
* **CRIAR 100% próprio:** O cérebro neural (`VillagerBrain`), a persistência SQLite/Nitrite, a interface de Voz (`Vosk`), o loteamento por chunks/estandartes, os construtores automáticos, a logística do Armazém, o Mapa-Múndi e os Esquadrões comandados por Chifres de Cabra.
