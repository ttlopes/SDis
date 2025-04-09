# 🕒 Fundamentos de Tempo em Sistemas Distribuídos

## 📏 Medir e Comparar Tempo

-   O tempo é essencial em SDs para:
    -   Protocolos de autenticação
    -   Sistemas de ficheiros (timestamps)
    -   Timeouts e medidas de desempenho

---

## 🧠 Motivação

### Exemplo: comando `make`

-   Num sistema local: funciona corretamente ao comparar timestamps
-   Num SD com relógios desalinhados: pode falhar ao detetar alterações

---

## ❓ O Problema

> É possível sincronizar todos os relógios num sistema distribuído?

Para responder, precisamos de um:

## 🧰 Modelo de Interação

### 🧮 Pressupostos sobre o canal de comunicação

-   Latência (espera, transmissão, processamento)
-   Largura de banda
-   Ordem das mensagens
-   Repetição de mensagens
-   Jitter (variação de tempos)
-   Desvio dos relógios locais

### 🕓 Sistemas Síncronos vs Assíncronos

-   **Síncrono**: limites conhecidos de tempo e desvio
-   **Assíncrono**: limites desconhecidos
-   Soluções para sistemas **assíncronos** funcionam também em **síncronos**

---

## ⏱️ Relógios Físicos

### 🎛️ Funcionamento

-   Cada máquina tem um relógio local
-   Idealmente sincronizados com o UTC

### ⚠️ Problemas

-   **Drift**: relógios divergem com o tempo
-   **Skew**: valores diferentes a cada instante
-   Pode parecer que uma mensagem foi recebida **antes** de ser enviada

---

## 🔧 Sincronização de Relógios

-   Objetivo: evitar que as diferenças entre relógios cresçam demasiado
-   Troca de mensagens periódicas entre nós

### 🔁 Fases da sincronização

1. Estimar valor dos relógios remotos
2. Ajustar relógio local com base nessa estimativa

---

## 🌐 Tipos de Sincronização

### 📎 Fases da sincronização de relógios

1. 🕵️ Estimar o valor dos relógios de outros processos
2. 🔧 Ajustar o relógio local com base nessa estimativa

---

### 🌍 Sincronização Externa

#### 📡 Como funciona

-   O tempo é **difundido por sinais de rádio terrestres** ou **sinais de satélite** (ex: GPS)
-   Os computadores podem ter **receptores de tempo** para sincronizar os seus relógios com o UTC
-   ⏱️ **Precisão dos sinais**:
    -   Rádio (estações terrestres): entre **0.1 ms e 10 ms**
    -   Satélites (GPS): cerca de **1 µs**

#### ❌ Sem tolerância a faltas

-   Um único processo tem acesso ao **UTC**
-   Os outros processos lêem o relógio desse processo
-   Todos ajustam os seus relógios para alinhar com a fonte externa

#### ✅ Com tolerância a faltas

-   Assume-se que **até f processos podem falhar** (ex: fornecer tempo errado), mas todos fornecem o **mesmo valor para todos os nós**
-   Configura-se um conjunto de **2f+1 processos com acesso ao UTC**
-   Cada processo lê o tempo dos 2f+1
-   Descartam-se os **f maiores e f menores valores**
-   Usa-se o **valor intermédio** (valor "do meio") como referência

📌 Nota: este é apenas um exemplo de estratégia — existem outras abordagens alternativas.

---

### 🔁 Sincronização Interna

#### ⚙️ Sem tolerância a faltas

-   Cada processo lê o relógio de todos os outros
-   Aplica uma **função (ex: média)** a todos os valores, incluindo o seu
-   Ajusta (atrasa ou adianta) o seu relógio para alinhar com o resultado da função

#### 🛡️ Com tolerância a faltas

-   Cada processo lê os valores dos outros relógios
-   **Descarta alguns valores suspeitos** (ex: outliers)
-   Aplica uma função (como a média) aos valores restantes e ao seu
-   Ajusta o seu relógio com base nesse resultado

---

## 🕵️‍♂️ Leitura de Relógios Remotos

### 🧪 Estimativa simples (modelo do Cristian)

1. `t1`: `p1` envia pedido a `p2`
2. `t2`: `p2` recebe o pedido, lê o valor X do seu relogio e responde com o seu tempo `X` (como estes 3 passos são muito rápidos, assumimos que acontecem no mesmo instante)
3. `t3`: `p1` recebe `x` e estima: `X + (t3 - t2)`

📌 O problema: `p1` **não sabe exatamente** quanto tempo a mensagem demorou  
➡️ **Resultado**: os relógios **nunca estarão perfeitamente sincronizados**

---

## 🧮 Algoritmos de Sincronização

`RTT` -> **Round-Trip Time**

### ⏱️ Algoritmo de Cristian (1989)

-   Cliente pergunta ao servidor o tempo
-   Estimativa: `C(t) = t + RTT/2`

### 📏 Precisão

-   Erro máximo: `±(RTT - 2 * min) / 2`

### 📘 Exemplo

-   RTT = 20ms → precisão = ±10ms
-   Se `min = 8ms` → nova precisão = ±2ms

---

### 👑 Algoritmo de Berkeley

-   Sincronização **interna**
-   O master:
    1. Pergunta o tempo a todos
    2. Corrige valores usando Cristian
    3. Calcula média e envia ajuste a cada nó
-   Nós com desvios altos são ignorados

---

## 🌐 NTP (Network Time Protocol)

-   Serviço global de sincronização com **UTC**
-   Altamente **tolerante a falhas** e **escalável**
-   Utilizado por milhões de dispositivos para manter os relógios sincronizados

### 🔄 Adaptação dinâmica à rede

-   A rede de NTP é **resiliente e adaptável** a falhas
-   Se um **servidor primário** perder ligação à fonte UTC:
    -   🔁 Assume o papel de **servidor secundário**
-   Se um **servidor secundário** perder ligação ao seu primário:
    -   🔀 Pode **alternar** e sincronizar com outro servidor primário

### 🛰️ Modos de funcionamento

-   **Multicast**: servidor LAN envia a todos
-   **Invocação remota**: cliente pergunta ao servidor (como Cristian)
-   **Simétrica**: pares de servidores trocam mensagens com timestamps

Em todos estes casos é usado `UDP`.

### 📦 Mensagens trocadas entre `Pares NTP` incluem:

-   Tempo de envio atual
-   Tempo de receção da última mensagem

📌 Cálculo do tempo de rede:  
`Tnet = (Ti - Ti-3) - (Ti-1 - Ti-2)`

---

## 🧱 Limitações dos Relógios Físicos

-   Todos os algoritmos estão sujeitos a **erro**
-   Nalguns sistemas, em que a rede pode estar sobrecarregada, este erro pode ser grande.
-   Relógios físicos nunca são **perfeitamente fiáveis**
-   A causalidade pode ser violada (ex: mensagem recebida "antes" de ser enviada)

---

## ⌛ Mas precisamos mesmo de tempo real?

Em muitos casos, só queremos **preservar a ordem dos eventos**, não o tempo absoluto.  
➡️ Podemos usar **relógios lógicos**

---

# 🧠 Relógios Lógicos

## 🔗 Aconteceu-antes (→)

-   `e1 → e2` se:
    -   Estão no mesmo processo e `e1` ocorre antes
    -   `e1` envia uma mensagem e `e2` a recebe
-   Relação é **transitiva** e define **ordem causal**

---

## 🔢 Relógios de Lamport

-   Cada processo `pi` tem um contador `Li`
-   **LC1**: incrementa `Li` antes de qualquer evento
-   **LC2a**: envia valor `t = Li` com cada mensagem
-   **LC2b**: ao receber `(m, t)`, faz `Li = max(Li, t) + 1`

### ❗ Limitações

-   `C(e) < C(e')` não garante que `e → e'`
-   `C(e) = C(e')` não implica que são o mesmo evento

Estas limitações existem nos **relógios de Lamport** e nos **relógios físicos**.

---

## 🧮 Relógios Vectoriais

São baseados nos relógios lógicos de Lamport.

-   Cada processo mantém um vetor `Vi` com um contador para **cada processo** e usam **pilhas**

### ⛓️ Regras:

-   **VC1**: `Vi[j] = 0` inicialmente
-   **VC2**: antes de evento local, `Vi[i]++`
-   **VC3**: envia vetor com mensagem e tempo
-   **VC4**: ao receber, `Vi[j] = max(Vi[j], t[j])` para todos `j` e depois faz `VC2`

### 📐 Comparação

-   `V ≤ V'` ⇔ `V[j] <= V'[j]` para todos `j = 1, 2, ..., N`
-   `V < V'` ⇔ `V ≤ V'` e `V ≠ V'`

#### 📎 Relação com causalidade

-   `e → e'` ⟺ `V(e) < V(e')`
-   `e || e'` (eventos concorrentes) ⟺ **nenhuma** das seguintes se verifica:
    -   `V(e) < V(e')`
    -   `V(e') < V(e)`
    -   `V(e) = V(e')`

➡️ Ou seja, se `V(e)` e `V(e')` **não são comparáveis**, os eventos são **concorrentes**.

---

## 🧠 Aplicações práticas

-   Preservar ordem causal em logs, backups, sistemas distribuídos
-   Resolver conflitos (ex: merges no Git)

---
