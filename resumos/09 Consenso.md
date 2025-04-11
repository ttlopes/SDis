# 🧠 Consenso em Sistemas Distribuídos

## 📌 O que é o Consenso?

-   Objetivo: obter **acordo entre processos** sobre um dado valor
-   Cada processo propõe um valor (input)
-   Todos os processos decidem o **mesmo valor (output)**
-   Valor decidido deve ser **um dos valores propostos** — ❌ sem valor por omissão 📌

-   Pode ser **qualquer** dos valores propostos:
    -   ❌ Não tem de ser o mais popular
    -   ❌ Não há valores “melhores” que outros
    -   ❌ Nem processos com mais peso ou prioridade

> 📌 Isto impede soluções que **ignoram os valores propostos** e decidem sempre o mesmo valor “por defeito”

---

## 🔒 Propriedades do Consenso

-   ✅ **Terminação**: todos os processos corretos **eventualmente decidem**
-   🤝 **Acordo (uniforme)**: se dois processos decidem, **decidem o mesmo valor**
-   🔐 **Integridade**: o valor decidido foi proposto por **algum processo**

---

## ❌ Impossibilidade do Consenso (FLP)

-   O consenso **não tem solução geral** em sistemas **assíncronos com falhas**
-   Este resultado de impossibilidade é conhecido como **Teorema FLP** (Fischer, Lynch, Paterson)

---

## ✅ Soluções possíveis

-   🔁 **Sistemas síncronos**:
    -   Permitem construir **detectores de falhas perfeitos**
-   🔁 **Sistemas parcialmente assíncronos**:
    -   Com detectores de falhas “eventualmente perfeitos”
    -   Exemplo: algoritmo **Paxos** (criado por Lamport mas fora da matéria de SD)

---

## 🌊 Algoritmo FloodSet (para sistemas síncronos)

-   Cada processo mantém um conjunto `S` com os valores recebidos
-   Em cada ronda:
    -   Envia `S` a todos os outros
    -   Junta os valores recebidos ao seu próprio conjunto
-   Após **f + 1 rondas** (f = máx. processos que podem falhar):
    -   Cada processo escolhe um valor de `S` e decide

📌 Se um processo **não recebe** mensagem de outro numa ronda, assume que esse processo **falhou**

## 🌊 Algoritmo FloodSet (sistemas síncronos)

-   Cada processo mantém um conjunto `S` com os valores recebidos
-   Em cada turno:
    -   Envia `S` aos outros
    -   Junta os valores recebidos ao seu conjunto
-   Após **f + 1 turnos** (f = nº máximo de falhas): decide um valor de `S`

📌 Assumindo sistema **síncrono**:

-   Se `pᵢ` não recebe de `pⱼ` no turno `n`, assume que `pⱼ` **falhou**

🛠️ Pode ser adaptado para usar um **detetor de falhas perfeito**:

-   Só avança se recebeu de `pⱼ` ou se o detetor o marcar como falhado

⚡ Sem falhas, pode terminar em **menos turnos**

---

## 🔁 Consenso e Replicação

Problemas que requerem consenso:

-   📜 Ordem total de mensagens
-   🧠 Máquina de estados replicada
-   👑 Primário-secundário
-   👥 Sincronia na vista

---

## 💬 Difusão Atómica com Consenso

-   Cada entrega atómica exige uma **nova instância de consenso**
-   Cada instância decide o **conjunto de mensagens a entregar**

### 🧱 Algoritmo simplificado

```pseudo
Init:
  por_ordenar = {}
  ordenadas = {}
  num_seq = 0

Quando AB.send(m):
  RB.send(m)

Quando RB.deliver(m):
  por_ordenar = por_ordenar ∪ {m}

Loop:
  espera até por_ordenar \ ordenadas ≠ {}
  num_seq += 1
  Consensus[num_seq].propose(por_ordenar \ ordenadas)
  espera até Consensus[num_seq].decide(proximas)
  ordenadas += proximas
  AB.deliver(m) para cada m em proximas (ordem determinista)
```

---

## 🤖 Máquina de Estados Replicada

-   Clientes usam **difusão atómica** para enviar pedidos
-   Réplicas:
    -   Recebem todas **o mesmo conjunto de pedidos**
    -   Pela **mesma ordem**
    -   Executam pedidos de forma **determinista**
-   Garante que todas mantêm o **mesmo estado**
-   Cada réplica responde ao cliente de forma idêntica

---

## 👑 Primário-Secundário com Consenso

-   Substitui **detetor de falhas** por uso de **consenso**
-   O primário propõe execução de pedidos
-   O consenso decide **qual pedido executar** e em **que ordem**

### 🧱 Lógica simplificada

```pseudo
Quando sou líder:
  escolho pedido pendente
  executo localmente
  envio proposta com estado e resposta via RB

Quando RB.deliver(proposta):
  guardo proposta
  se líder e proposta é válida:
    proponho via consenso[num_seq + 1]

Quando consenso decide:
  aplico estado
  marco pedido como executado
  envio resposta ao cliente
```

---

## 👥 Sincronia na Vista com Consenso

-   A transição da vista `Vi` para `Vi+1` requer um **acordo entre os processos (consenso)**
-   Cada processo mantém `hᵢ`: mensagens que já entregou em `Vi`

### 🧱 Passos:

1. Envia `"view-change"` para os restantes processos de `Vi`
    - Contém saídas e entradas (podem ser vazias)
2. Ao receber `"view-change"`, para de entregar novas mensagens
    - Cria `Vi+1 = Vi - saídas + entradas`
    - Inicializa `hⱼ = null` para cada processo `pⱼ` da `Vi`
    - Envia `hᵢ` para todos os processos de `Vi+1`
3. Recolhe os `hⱼ` dos outros processos
    - Se algum `hⱼ` não chegar, e `pⱼ` for suspeito de falha → define `hⱼ = {}`
4. Quando tem todos os `hⱼ` (recebidos ou assumidos), define:
    - `M-Vi = ⋃ hⱼ` (mensagens a entregar em falta)
5. Propõe via consenso o tuplo `<Vi+1, M-Vi>`
6. O resultado define:
    - A nova vista `Vi+1`
    - O conjunto de mensagens finais a entregar em `Vi`

📌 Garante que **todos os processos da nova vista**:

-   Têm **vista consistente**
-   E partilham o **mesmo histórico de mensagens entregues**

---
