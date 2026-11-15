# GDD — Realms & Horns (Game Design Document)

## 1. Conceito Central (High Concept)
**Realms & Horns** transforma o Minecraft em um simulador vivo de civilizações medievais com fantasia arcana. Vilas deixam de ser estruturas estáticas e passam a ser organismos econômicos e sociais operados por villagers dotados de redes neurais locais, memórias persistentes, relações familiares e ambições políticas. O jogador pode reivindicar uma vila com seu estandarte pessoal, expandir seu território até formar um reino de 100 chunks, gerenciar cadeias produtivas e liderar exércitos em primeira pessoa usando **Chifres de Cabra**.

---

## 2. Território, Estandartes e Progressão de Assentamentos

### 2.1. Sistema de Estandarte Pessoal
* No primeiro login no mundo, o jogador visualiza uma interface dedicada para desenhar seu **Estandarte de Linhagem** (padrões, cores e brasão).
* Para reivindicar uma vila neutra (ou fundar um novo assentamento), o jogador interage com o **Pedestal Central da Vila** e finca seu estandarte.

### 2.2. Escala Territorial (Base: Chunks de 16×16 Blocos)
| Nível do Assentamento | Limite Territorial | Requisitos Principais | Capacidade Populacional |
| :--- | :--- | :--- | :--- |
| **Vila Pequena** | **15 chunks** | Estandarte reivindicado + Armazém Central | Até 20 villagers |
| **Capital** | **40 chunks** | Muralha nível 1 + Restaurante + Templo + 25 habitantes | Até 60 villagers |
| **Reino** | **100 chunks** | Castelo construído + Guilda dos Cartógrafos + 60 habitantes | Até 150 villagers |

### 2.3. Autonomia Sem Jogador e Fog of War Político
* Vilas não reivindicadas possuem um **Villager Político (Ancião/Prefeito)** que gerencia decisões básicas.
* **Isolamento Inicial:** Vilas sem jogador desconhecem a existência umas das outras e evoluem em ritmo reduzido (15% da velocidade normal).
* **Descoberta:** Quando um jogador ou um **Villager Explorador** localiza outra vila, o contato diplomático é estabelecido no **Mapa-Múndi**, habilitando comércio, alianças ou declarações de guerra.

---

## 3. O Cérebro do Villager (Arquitetura Cognitiva em 5 Pilares)

Cada villager possui identidade única, nome próprio e um cérebro composto por:

1. **Objetivo (`Objective`):**
   * Define a tarefa imediata e o **porquê** sistêmico.
   * *Exemplo:* `"Cortando carvalho porque o construtor Junior abriu requisição de 32 tábuas no Armazém para a Casa Nível 2."`
   * Ao clicar no villager ou conversar com ele, o jogador pode consultar exatamente o que ele está fazendo e o motivo.
2. **Relações (`Relations`):**
   * Registra quem o villager conhece (outros villagers e jogadores), nível de afinidade (`-100` a `+100`) e o resumo das últimas conversas/fofocas.
   * Influencia cooperação no trabalho, formação de casais e lealdade ao governante.
3. **Rotina (`Routine`):**
   * Relógio biológico e social:
     * **06:00 – 12:00:** Turno matinal de trabalho.
     * **12:00 – 13:30:** Almoço no **Restaurante** (interação social e troca de informações).
     * **13:30 – 17:30:** Turno vespertino de trabalho.
     * **17:30 – 19:00:** Oração no **Templo** (para perfis devotos) ou lazer na praça/restaurante.
     * **20:00 – 06:00:** Descanso em sua residência atribuída.
4. **Personalidade (`Personality`):**
   * Cinco eixos contínuos (`0.0` a `1.0`): **Coragem**, **Ganância**, **Sociabilidade**, **Devoção** e **Diligência**.
   * Determina o estilo de fala no chat/voz, predisposição para se alistar no exército, reação a impostos e resistência ao medo em batalha.
5. **Outros (`Context & Survival`):**
   * **Heatmap Espacial:** Memória de locais ricos em minério/madeira e zonas letais (onde houve ataques de mobs ou batalhas).
   * **Necessidades & Moral:** Fome, exaustão e moral coletiva.
   * **Proficiência (XP):** Quanto mais tempo um villager exerce sua profissão sem morrer, mais rápido trabalha e melhores itens/feitiços desbloqueia.

### 3.1. Interação por Chat e Voz
* O jogador pode digitar no chat próximo a um villager ou segurar a tecla de **Voz (Push-to-Talk)**.
* A fala é transcrita localmente para texto e enviada ao cérebro do villager, que interpreta a intenção (saudação, pergunta sobre objetivo, ordem direta, elogio/ameaça) e responde de acordo com sua **Personalidade**, **Relações** e estado atual da vila.

---

## 4. Profissões e Cadeia Produtiva (Ordem de Desenvolvimento)

As profissões são implementadas sequencialmente, conectadas pelo **Armazém Central**:

### Bloco 1 — Extração e Infraestrutura
1. **Lenhador & Minerador (Coletores):** Extraem madeira, pedra e minérios nas áreas demarcadas (ou exploradas) e depositam tudo nos baús indexados do **Armazém Central**.
2. **Construtor Civil:** Lê as plantas arquitetônicas (`.nbt`), retira os materiais necessários do Armazém e ergue/evolui prédios por níveis:
   * *Cabana (Nível 1) -> Casa (Nível 2) -> Mansão (Nível 3) -> Castelo (Nível 4)*, além de *Armazém, Restaurante, Templo, Forja e Muralhas*.
3. **Construtor Viário (Engenheiro de Caminhos):** Detecta novas construções, executa terraplanagem suave no terreno e conecta todas as portas ao centro da vila por estradas pavimentadas e iluminadas.

### Bloco 2 — Alimentação e Vida Social
4. **Fazendeiro:** Cultiva trigo, batata, cenoura e beterraba; fabrica **ração animal** e assa **pães**.
5. **Pecuarista:** Gerencia cercados de gado, ovelhas, porcos e galinhas; alimenta, reproduz e realiza o abate controlado para abastecer o Armazém com carne e couro.
6. **Cozinheiro:** Opera o **Restaurante**. Retira ingredientes crus do Armazém, prepara refeições completas e serve os cidadãos nos horários de pico. O Restaurante é o principal gatilho de socialização e atualização de `Relações`.

### Bloco 3 — Política e Exploração
7. **Político (Prefeito/Conselheiro):** Gerencia prioridades de construção, coleta impostos simbólicos para o tesouro da cidade e representa vilas autônomas.
8. **Explorador / Cartógrafo:** Viaja para fora dos limites dos chunks da vila, mapeia cavernas, biomas e ruínas, descobre vilas vizinhas, atua como **mensageiro diplomático** e deposita mapas físicos na **Mesa de Mapa-Múndi** da Guilda (revelando o território para o jogador).

### Bloco 4 — Forja, Magia e Fé
9. **Ferreiro:** Transforma lingotes do Armazém em armas e armaduras medievais (integração com *Epic Knights*).
10. **Encantador:** Trabalha na Torre Arcana aplicando encantamentos nos equipamentos destinados ao exército.
11. **Clérigo:** Atua no **Templo** durante a rotina de paz (aumentando a moral e reduzindo o luto) e marcha junto aos exércitos em guerra como unidade de suporte/cura.

---

## 5. Ciclo Vital: Casamentos, Nascimentos e Funerais

* **Casamentos & Natalidade:** Quando dois villagers adultos atingem alta afinidade (`maior que 75` em `Relações`) e há uma cama livre em uma residência da vila, ocorre um evento público de **Casamento** na praça/templo. Após alguns dias no jogo, nasce um novo villager criança que herda traços de personalidade dos pais.
* **Morte Permanente & Funerais:** A morte de qualquer villager é **definitiva**.
  * **Funeral Individual:** Ocorre no cemitério/templo da vila, reduzindo temporariamente a produtividade dos amigos próximos e gerando memórias de luto.
  * **Funeral Coletivo de Guerra:** Após o término de uma batalha, os sobreviventes realizam uma cerimônia única para todos os soldados caídos. A perda de veteranos impacta fortemente a moral do reino.

---

## 6. Sistema de Guerra, Comandos por Chifres e Cerco

### 6.1. Composição do Exército e Generais
O exército de um reino é dividido em **Esquadrões**, cada um liderado por um **Villager General**:
* **Infantaria:** Linha de frente com espadas, machados e escudos.
* **Arqueiros:** Ataque à distância e defesa de muralhas.
* **Cavalaria:** Villagers montados em cavalos de guerra para flanqueamento rápido.
* **Magos:** Conjuradores ofensivos (integração com *Iron's Spells 'n Spellbooks*).
* **Clérigos:** Unidades de retaguarda que conjuram cura em área nas tropas aliadas.

### 6.2. Controle Tático em 1ª Pessoa via Chifres de Cabra
Enquanto luta diretamente no campo de batalha ao lado de suas tropas, o jogador utiliza **Chifres de Cabra** afinados para emitir ordens instantâneas aos Generais num raio de 64 blocos:
* **Chifre de Bronze (Som Grave — *Ponder*):** **Avançar / Investida Total** contra o alvo marcado.
* **Chifre de Prata (Som Agudo — *Call*):** **Recuar / Reagrupar** próximo à posição do jogador.
* **Chifre de Guerra (Som Duplo — *Seek*):** **Cercar / Flanquear** (Cavalaria e Infantaria contornam os flancos inimigos).
* **Chifre de Ferro (Som Longo — *Sing*):** **Manter Posição / Formação Defensiva** (escudos erguidos e arqueiros/magos protegidos no centro).

### 6.3. Moral e Queda de Líder
* Cada exército possui uma barra de **Moral (0 a 100%)**.
* Se um **Villager General** é abatido em combate, um som dramático ecoa para todos os jogadores na região e surge o aviso na tela: **`[Líder Derrotado]`**.
* A tropa que perdeu o general sofre queda brusca de moral (podendo entrar em debandada), enquanto o exército vencedor recebe bônus temporário de ataque e moral.

### 6.4. Regra de Cerco (Anti-Frustração)
* Para proteger horas de construção dos jogadores, **apenas blocos de Muralha e Portões** podem ser destruídos durante uma guerra.
* Casas, armazéns, restaurantes e templos são indestrutíveis durante o cerco.
* **Condição de Vitória:** Invadir a cidade e capturar/interagir com o **Estandarte Central (Trono)** por 30 segundos contínuos sem inimigos na zona de captura.
