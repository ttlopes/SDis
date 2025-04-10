# 🛡️ Tolerância a Faltas

## 🔄 Modelo de Interação

### 🔁 Sistemas Síncronos

-   📬 As mensagens chegam no destino dentro de um **tempo máximo conhecido**
-   ⏱️ As operações têm **tempos de execução previsíveis** (mínimos e máximos)
-   🕓 O desvio dos relógios locais é **limitado e conhecido**
-   ✅ Todos estes limites são assumidos como conhecidos

📌 **Nota**: Se algum destes limites **não for conhecido**, o sistema deve ser considerado **assíncrono**

### 🔁 Sistemas Assíncronos

-   ❌ **Não há garantias** sobre:
    -   Tempo de transmissão de mensagens
    -   Duração das operações
    -   Sincronização dos relógios
-   ➡️ Mais **realista** para sistemas distribuídos reais
-   ⚠️ **Impossível detetar falhas com certeza** — não se consegue distinguir entre uma falha e um simples atraso

---

## ⚠️ Modelo de Faltas

-   É necessário identificar quais são as **faltas expectáveis** no sistema
-   Em seguida, decidir:
    -   Quais as faltas que o sistema irá **recuperar**
    -   Quais as que **não serão toleradas**
-   📊 A **taxa de cobertura** é a razão entre:
    -   O número de faltas que podem ser recuperadas
    -   E o total de faltas previsíveis

⚠️ As faltas que originam **erros sem possibilidade de tratamento** resultam em **falhas catastróficas** do sistema

---

### 🧩 Modelo de Faltas num Sistema Distribuído

-   O modelo de faltas é muito mais **complexo** do que em sistemas centralizados
-   Várias **componentes podem falhar**, incluindo:
    -   🌐 Faltas na comunicação (ex: perdas de mensagens, atrasos)
    -   🖥️ Faltas nos nós (ex: falhas de hardware ou energia)
    -   🧠 Faltas nos processadores ou no sistema operativo
    -   🧑‍💻 Faltas nos processos (clientes ou servidores)
    -   💾 Faltas nos meios de **armazenamento persistente**

---

## 🔧 Tipos de Faltas

### 🤐 Faltas Silenciosas

-   A componente **deixa de responder**
-   Pode ser:
    -   **Fail-stop**: falha detetável (o processo para completamente)
    -   **Crash**: falha não detetável (o processo para sem aviso)

---

### 🤯 Faltas Arbitrárias (Bizantinas)

-   Comportamentos **imprevisíveis ou maliciosos**
-   ⚠️ Consideradas o **pior caso possível**, mas úteis para modelar ataques

#### De um processo:

-   Responde incorretamente a estímulos
-   Responde mesmo sem ter recebido estímulo
-   Não responde quando deveria

#### Do canal de comunicação:

-   Mensagem chega com conteúdo corrompido
-   Entrega de mensagens **inexistentes ou duplicadas**
-   Não entrega a mensagem

📌 Faltas bizantinas no canal são **raras**, pois:

-   Protocolos de comunicação geralmente **detetam** estas situações
-   Podem usar **checksums**, **ACKs**, **retransmissões** e outras técnicas

---

### 💬 Nota sobre suposições

#### Por omissão, assume-se falta silenciosa

-   Por simplificação, assume-se frequentemente que as faltas são **silenciosas**
-   Mas essa **assunção nem sempre se verifica** na realidade

#### Faltas tipicamente não consideradas na recuperação:

-   **Faltas densas** (várias falhas simultâneas acumuladas)
-   **Faltas bizantinas**

⚠️ Se o modelo de faltas assumido **não refletir a realidade**, o sistema pode **não cumprir a sua especificação**, mesmo tendo sido desenhado para ser "tolerante a falhas"

---

## 🧵 Faltas por componente

### 📦 Faltas silenciosas dos processos

-   Processo deixa de responder a estímulos

### 🌐 Faltas silenciosas do canal

-   📨 **Send-omission**

    -   A mensagem **perde-se entre o processo emissor e o buffer de saída da rede**

-   🌐 **Channel-omission**

    -   A mensagem **perde-se durante a transmissão**, entre os buffers de saída e de entrada

-   📥 **Receive-omission**
    -   A mensagem **chega ao buffer de entrada do recetor**, mas **perde-se antes de ser entregue ao processo**

---

## ⚠️ Faltas não consideradas (por omissão)

-   🧱 **Falta densa**: acumulação de várias faltas toleráveis → falha total
-   🌀 **Falta bizantina**: comportamento diferente para diferentes destinos

---

## 🚨 Deteção de Faltas

### 🔍 Em sistemas síncronos

-   Assume-se **latência máxima** conhecida
-   Detetar falhas remotamente é possível

### ❌ Em sistemas assíncronos

-   💣 Não é possível distinguir entre latência e falha
-   ❗ Não se pode detetar falha remota com certeza

---

## 📏 Métricas de Tolerância a Faltas

> 🕒 **MTTF** – _Mean Time To Failure_  
> 🔁 **MTBF** – _Mean Time Between Failures_  
> 🛠️ **MTTR** – _Mean Time To Repair_

### 🕒 Fiabilidade

-   **MTTF**: tempo médio até à primeira falha
    -   Usado em **sistemas não reparáveis**
-   **MTBF**: tempo médio entre duas falhas consecutivas
    -   Mede o tempo médio de **funcionamento correto** entre falhas
    -   Usado em **sistemas reparáveis**

### 🛠️ Reparação

-   **MTTR**: tempo médio necessário para **detetar, diagnosticar e corrigir** uma falha

### 📈 Disponibilidade

> Fórmula:  
> `Disponibilidade = MTBF / (MTBF + MTTR)`

-   Mede a **proporção de tempo em que o sistema está operacional**
-   Depende da **frequência das falhas** e da **eficiência da reparação**

### 🧮 Exemplo prático

-   9 horas de funcionamento, 4 falhas, total de 1 hora de inatividade:
    -   `MTBF = (9 - 1) / 4 = 2h`
    -   `MTTR = 1 / 4 = 15 min`
    -   `Disponibilidade = 2 / (2 + 0.25) = 88%`

### 🚀 Melhoria da Disponibilidade

-   **MTBF** representa a **fiabilidade** do sistema  
    ✅ Quanto maior, melhor — menos falhas
-   **MTTR** reflete a **eficiência da recuperação**  
    ✅ Quanto menor, melhor — reparações mais rápidas

📌 Para melhorar a disponibilidade:

-   Aumentar a **qualidade do serviço** (menos falhas → ↑MTBF)
-   Otimizar a **reparação e manutenção** (reparar mais depressa → ↓MTTR)

🎯 Objetivo: aproximar a disponibilidade dos **100%**  
⚠️ Mas sabemos que **100% nunca é atingível na prática**

---

## 🧱 Classes de Disponibilidade

📌 **D = Disponibilidade** (também chamada de "**número de noves**")  
📐 A **Classe de Disponibilidade** pode ser calculada como:

> `Classe = log10 [1 / (1 - D)]`

Por exemplo:

-   99.9% → Classe ≈ 3 (3 noves)
-   99.999% → Classe ≈ 5 (5 noves)

---

### 📊 Tabela de equivalência

| Nível de Disponibilidade | Disponibilidade (%) | Indisponibilidade anual |
| ------------------------ | ------------------- | ----------------------- |
| Comercial ou standard    | 99.5                | 43.8 horas              |
| Highly available         | 99.9                | 8.75 horas              |
| Fault resilient          | 99.99               | 53 minutos              |
| Fault tolerant           | 99.999              | 5 minutos               |
| Continuous               | 100                 | 0 minutos               |

---

### 📋 Outra Tabela de Classificação (por classe)

| Classe | Disponibilidade | Indisponibilidade (min/ano) | Designação                 |
| ------ | --------------- | --------------------------- | -------------------------- |
| 1      | 90%             | 52 560                      | Não gerido                 |
| 2      | 99%             | 5 256                       | Gerido                     |
| 3      | 99.9%           | 526                         | Bem gerido                 |
| 4      | 99.99%          | 53                          | Tolerante a faltas         |
| 5      | 99.999%         | 5                           | Alta disponibilidade       |
| 6      | 99.9999%        | 0.5                         | Muito alta disponibilidade |
| 7      | 99.99999%       | 0.05                        | Ultra disponibilidade      |

---

## 🔁 Técnicas de Recuperação

### ⏮️ Recuperação para trás

-   Guarda periodicamente o estado (**checkpoint**)
-   Se falhar: reinicia a partir do último checkpoint
-   ⚠️ Os estados guardados devem estar **mutuamente coerentes** entre processos

### ⏩ Recuperação para a frente

-   Mantém **várias réplicas** ativas
-   Se uma falhar, usa-se outra
-   ⚠️ O estado das réplicas sobreviventes deve estar **coerente**

> ⏮️ **Recuperação para trás**  
> [Checkpoint 1] → [Checkpoint 2] → ❌ Falha → 🔁 Retoma a partir do último checkpoint
>
> ⏩ **Recuperação para a frente**  
> [Réplica A] ✅ [Réplica B] ❌ [Réplica C] ✅ → 🔁 Serviço continua com réplicas ativas

---

# 🧬 Replicação Tolerante a Faltas

## 🎯 Objetivos

-   Garantir **linearizabilidade (coerência forte)**
-   Tolerar falhas de processos ou comunicação

---

## 👑 Primário-Secundário (esboço)

-   Um processo é eleito **primário**
-   Se falhar → é eleito novo primário
-   Cliente envia pedidos ao primário:
    1. Executa o pedido
    2. Propaga estado para secundários e aguarda confirmações
    3. Responde ao cliente
-   Depois, processa o próximo pedido da fila

---

## 🧠 Replicação de Máquina de Estados (RME)

-   Cliente envia pedidos a **todas as réplicas**
-   Todos os pedidos são **ordenados por ordem total**
-   Todas executam os pedidos **na mesma ordem**
-   Requer operações **deterministas**

---

## 📦 Abstrações usadas para construção de sistemas replicados

Estas abstrações ajudam a **garantir propriedades fortes de comunicação e consistência**, mesmo na presença de falhas.

---

### 📬 Canais Perfeitos

-   Garante **entrega ponto a ponto**, **ordenada** e **sem perdas**, desde que o emissor e recetor sejam corretos
-   Como implementar:
    -   Reenvio contínuo da mensagem até receber **ACK**
    -   Utilização de **IDs** para evitar entregas duplicadas ou fora de ordem
-   Úteis como base para construir **protocolos mais complexos**

---

### 📡 Difusão Fiável (Reliable Multicast)

> Permite enviar uma mensagem para um grupo com garantias de entrega fiável

-   Dois eventos básicos:
    -   `multicast(grupo, mensagem)`
    -   `deliver(mensagem)`

#### 🔁 Difusão Fiável Regular

-   ✅ Propriedades:

    -   **Validade**: se um processo correto envia, acabará por entregar
    -   **Não-duplicação**: cada mensagem é entregue no máximo uma vez
    -   **Não-criação**: só se entrega o que foi enviado por um processo correto
    -   **Acordo**: se um processo correto entrega, **todos os corretos** entregam

-   🧪 Algoritmo:

    -   O emissor envia a mensagem via **canais perfeitos**
    -   Quando um membro recebe, entrega à aplicação e **reenvia para o grupo**

-   ⚠️ Limitação:
    -   Se o **emissor falhar** antes de todos receberem, pode haver perda

#### 🛡️ Difusão Fiável Uniforme

-   🔒 Mais forte que a regular: garante **entrega uniforme entre os corretos**
-   ✅ Propriedades:

    -   Igual à regular **+** se um processo correto entrega, **todos ou nenhum** entregam

-   Útil quando a consistência total entre todos os processos é crítica (ex: **ordem total**)

### 📢 Difusão Atómica

> Variante de difusão fiável **com ordem total**

-   Garante que **todas as réplicas entregam a mensagem pela mesma ordem**
-   Base para **replicação de máquina de estados (RME)**
-   Requer mecanismos de **consenso ou sequenciador**

---

### 👥 Sincronia na Vista

> Garante **coerência entre membros de um grupo**, mesmo quando há mudanças de filiação, de forma a facilitar a tolerância a faltas

#### 🧩 Conceitos principais:

-   **Vista (view)**: conjunto de processos **ativos** que pertencem ao grupo
    -   Novos membros podem ser **adicionados dinamicamente**
    -   Um processo pode **sair voluntariamente** ou ser **expulso** se falhar
-   Vistas evoluem de forma **totalmente ordenada**

    -   Ex:
        -   `V1 = {p1, p2, p3}`
        -   `V2 = {p1, p2, p3, p4}`
        -   `V3 = {p2, p3, p4}`

-   Um processo é **considerado correto** na vista `Vi` se:
    -   Pertence a `Vi` **e também a `Vi+1`**

#### ✉️ Regras de entrega de mensagens:

-   Uma mensagem **enviada em Vi** é **entregue apenas em Vi**
-   Todos os processos corretos que passam de `Vi` para `Vi+1` **entregam exatamente as mesmas mensagens em `Vi`**

---

#### 🔄 Difusão fiável síncrona na vista

-   Se um processo correto envia `m` na vista `Vi`, então:
    -   `m` é entregue **a ele próprio**
    -   E **a todos os outros processos corretos de Vi**
-   Isto permite manter **consistência durante alterações de vista**

---

#### 🔁 Mudança de Vista

-   Durante a mudança de vista:
    -   As aplicações devem **pausar temporariamente o envio de mensagens**, de forma a que o conjunto de mensagens a entregar na vista seja finito
    -   É necessário executar um algoritmo de **coordenação** para:
        -   Decidir a **composição da nova vista**
        -   Determinar o **conjunto de mensagens a entregar antes da transição**

⚠️ Nota: Os **algoritmos de mudança de vista** não são abordados nesta cadeira

---

## ⚙️ Concretizar os Algoritmos

### 👑 Primário-Secundário

-   As réplicas usam **sincronia na vista** para lidar com falhas do primário
-   Quando o primário `p` falha, uma **nova vista** será eventualmente entregue sem `p`
-   Os restantes processos **elegem um novo primário**, por exemplo:
    -   O **primeiro processo da nova vista**
    -   Ou usando um **algoritmo de eleição de líder**
-   O novo primário anuncia o seu endereço num **serviço de nomes**, para que os clientes saibam onde enviar pedidos
-   Para propagar o estado aos secundários, o primário usa:
    -   **Difusão fiável uniforme síncrona na vista**
-   ✅ Só responde ao cliente depois de garantir que **todos os corretos** receberam o novo estado — garantindo **uniformidade**

### 🤖 Máquina de Estados (RME)

-   Todas as réplicas usam **sincronia na vista** para manter coerência
-   Os clientes enviam pedidos por **difusão atómica**
-   As réplicas recebem os pedidos na **mesma ordem** e executam-nos de forma **determinista**
-   Garante que todas mantêm o **mesmo estado**, mesmo com falhas

---

## 💬 Difusão Atómica

-   Variante fiável e com **ordem total**
-   Garante que todas as réplicas recebem os pedidos:
    -   Pela **mesma ordem**
    -   Ou **nenhuma recebe**

---

## 🔁 Ordem Total com Faltas

### 🧭 Sequenciador

-   Um **líder/sequenciador** define a ordem total das mensagens
-   Se o sequenciador **falhar**, podem surgir inconsistências:
    -   📬 Mensagens de dados que **ainda não tinham sido ordenadas**
    -   📥 Réplicas podem receber **mensagens do líder sobre mensagens que ainda não chegaram**
    -   🤔 Réplicas diferentes podem ter **visões distintas do estado do sistema**

⚠️ É necessário um **algoritmo de recuperação com consenso** para restaurar o estado consistente entre réplicas

### 🧩 Acordo Coletivo

-   Sofre dos mesmos problemas em caso de falha
-   Também requer mecanismo de consenso

---

### 🤖 Replicação de Máquina de Estados (concretização final)

-   Os **processos usam sincronia na vista** para lidar com alterações no grupo
-   Os **clientes enviam pedidos** através de **difusão atómica síncrona na vista** para todas as réplicas
-   Garante que **todas as réplicas corretas processam os pedidos na mesma ordem**, mantendo o estado consistente

---
