# 🧱 Replicação de Objetos Genéricos

## ❓ Qual a estrutura de dados replicada?

-   🗃️ **Registo** → já visto com o algoritmo **ABD**
-   📦 **Espaço de tuplos** → já visto com o algoritmo **Xu-Liskov**
-   🧩 **Qualquer estrutura de dados** com uma **interface remota genérica**

### ⚠️ Nota

-   Algoritmos para objetos genéricos **não são tão otimizados**
-   Não podem usar estratégias específicas (como versões ou quorum reduzido)
-   Por isso, são mais **genéricos mas menos eficientes**

---

## 🎯 Objetivo

-   Construir **algoritmos genéricos de replicação**
-   Garantir **linearizabilidade (coerência forte)**

---

## 🔄 Duas variantes principais

### 👑 Primário-Secundário

-   Um processo é eleito como **primário**
-   Os clientes enviam pedidos ao primário
-   Para cada pedido:
    1. O primário **executa** o pedido
    2. **Propaga o novo estado** para os secundários e aguarda confirmação de **todos**
    3. Só então **responde ao cliente**
-   Depois, processa o próximo pedido da fila

📌 Muito semelhante a um **servidor central com backup**

---

### 🧠 Replicação de Máquina de Estados (RME)

-   Os clientes enviam os pedidos para **todas as réplicas**
-   Todos os pedidos são processados e ordenados na **mesma ordem** (`ordem total`)
-   Cada réplica executa as mesmas operações, pela mesma ordem
-   Assumem-se operações **deterministas**
    -   Resultado depende apenas do estado + inputs
    -   Garante que todas as réplicas chegam ao mesmo estado
    -   No final todas as replicas ficarão idênticas

---

## 📦 Ordem Total — Como garantir?

### 📌 Dois algoritmos principais:

#### 1️⃣ Ordem total com **Sequenciador**

-   As mensagens são enviadas a todas as réplicas (difusão fiável)
-   Um processo é eleito **líder/sequenciador**:
    1. Decide a **ordem de entrega**
    2. Envia essa ordem às outras réplicas

#### 2️⃣ Ordem total por **Acordo Coletivo**

-   Baseado em **consenso entre réplicas**
-   Todas participam na decisão da ordem

---

## ⚖️ Comparação: Sequenciador vs Acordo Coletivo

### 📌 Cenário I: um único objeto replicado por um grupo

-   🧠 Sequenciador tende a ser mais **escalável** (centraliza decisão)

### 📌 Cenário II: múltiplos objetos, replicados em grupos de nós distintos

-   ⚠️ Um nó pode pertencer a mais do que um grupo (replica vários objetos)
-   ❌ Com sequenciador:
    -   Exige um **sequenciador global** → ponto de congestionamento
-   ✅ Com acordo coletivo:
    -   Funciona mesmo com grupos **diferentes** por objeto
    -   Cada grupo pode acordar a ordem **localmente**, sem coordenador global

➡️ **Acordo coletivo** é mais escalável neste cenário distribuído e heterogéneo.

---

## ⚔️ Comparação: Primário-Secundário vs RME

| Critério             | Primário-Secundário                          | RME (Máquina de Estados)                     |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| Tipo de operações    | Suporta **não deterministas**                | Requer operações **deterministas**           |
| Propagação de erros  | Um erro do líder é **replicado**             | Um erro numa réplica **não afeta as outras** |
| Forma de coordenação | Centralizada (no líder)                      | Totalmente distribuída                       |
| Tolerância a falhas  | Menor (se o líder falha, precisa de eleição) | Maior (cada réplica executa igual)           |

---
