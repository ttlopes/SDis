# 🧠 Coerência Fraca e Replicação Otimista

## 🧭 Teorema CAP

> É impossível garantir simultaneamente:

-   🔗 **Coerência** (Consistency)
-   ⚙️ **Disponibilidade** (Availability)
-   🌐 **Tolerância a Partições** (Partition tolerance)

📌 Num sistema distribuído, **apenas duas destas três propriedades** podem ser garantidas ao mesmo tempo.

---

## 🌊 Propagação Epidémica (Gossip)

-   Cada réplica contacta periodicamente outra e envia atualizações
-   As modificações espalham-se de forma **descentralizada e balanceada**
-   Também chamada de:
    -   **Propagação Rumorosa**
    -   **Gossip Architecture**
-   Oferece escalabilidade e resiliência, mas com **consistência fraca**

---

## 🤔 Problemas levantados

1. **Em que ordem aplicar as atualizações recebidas?**  
   ✅ Ordem **causal**, respeitando a relação “aconteceu-antes”

2. **Se um cliente consulta duas réplicas diferentes, que garantias oferecer?**  
   ✅ Que vê um estado **coerente com o histórico causal**

---

## 📏 Como garantir causalidade?

-   Utilizam-se **relógios vetoriais** para seguir dependências
-   Cada cliente mantém um vetor `prev` com as últimas versões vistas
-   Cada réplica responde com `new` → timestamp vetorial atualizado

---

## 🔄 Algoritmo: Interação Cliente–Réplica

1. Cliente envia pedido + `prev`
2. Réplica responde com resposta + `new`
3. Cliente atualiza o seu vetor `prev` com o novo timestamp

🧠 As réplicas **esperam até estarem atualizadas** antes de responder, para respeitar causalidade

---

## 🧮 Estado mantido por uma réplica

-   🕒 **Replica timestamp**: modificações locais aplicadas
-   📝 **Update log**: modificações recebidas mas não aplicadas
-   ⌛ **Value timestamp**: estado visível (executado)
-   📋 **Executed table**: evita execuções duplicadas
-   🔁 **Gossip messages**: comunicações periódicas com outras réplicas

---

## 📬 Pedidos

### 📖 Leitura

-   Se `prev ≤ value timestamp`, responde imediatamente
-   Caso contrário, espera até o estado estar suficientemente atualizado

### ✍️ Modificação

1. Verifica se já foi executada (duplicação)
2. Atualiza a sua entrada em `replicaTS`
3. Gera novo timestamp vetorial com base em `prev`
4. Junta ao log e responde ao cliente
5. Executa apenas quando `prev ≤ valueTS`

---

## 🔁 Propagação de modificações (Gossip)

-   Cada réplica contacta outra e envia as atualizações que **julga que o outro ainda não tem**
-   As modificações são:
    -   Enviadas por ordem
    -   Armazenadas se ainda tiverem dependências pendentes
    -   Executadas assim que forem **causalmente estáveis**

---

## 📚 Exemplo de arquitetura Gossip

### 🏦 Sistema com 2 réplicas (R0 e R1)

> `Alice` e `Bob` começam com o saldo nulo

-   A conta `S` faz duas transferências:
    -   R0 processa `S → Alice`
    -   R1 processa `S → Bob`
-   Inicialmente: ambos veem apenas a sua parte
-   Após propagação: ambas as réplicas têm o estado completo e consistente

---

## ⚠️ Problemas em leitura prematura

-   Se um cliente contacta R0 (atualizada) e depois R1 (atrasada), R1 pode:
    -   ❌ Responder com dados desatualizados
-   🧠 Por isso, R1 só deve responder **após satisfazer as dependências de `prev`**

---

## 🐢 Lazy Replication

-   Atualizações **não são propagadas imediatamente**
-   As réplicas podem estar em **estados temporariamente divergentes**
-   Existem algoritmos mais fortes para:
    -   💪 **Forced operations**
    -   ⚡ **Immediate operations**

➡️ Estes não são abordados nesta cadeira.

---

## 🌀 Bayou

-   Assumem que operações concorrentes são **comutativas**
    -   Ex: “mudar cor” + “alterar espessura”
-   Todas as escritas são aceites na réplica
-   Conflitos são resolvidos por **políticas específicas da aplicação**

### 🎯 Exemplo:

-   Preferir alterações feitas por professores em relação a alunos

📌 Algumas escritas anteriores podem ser **desfeitas**, conforme a política

⚠️ **Se as operações não forem comutativas**, devem ser usadas as primitivas **mais fortes**:

-   `forced operations`
-   `immediate operations`

---

## ✍️ Updates tentativos e commit

-   Uma operação começa como **tentativa**
    -   Pode ainda ser alterada
-   Torna-se **committed** quando a ordem fica definida
    -   Decidida por um **primário**
-   Réplicas podem ter de **reordenar operações** ao receber o commit

---
