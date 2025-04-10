# 🧺 Espaços de Tuplos Replicados

## 📌 Conceito: Espaço de Tuplos “Linda”

-   Um **espaço partilhado** onde se colocam e acedem **tuplos**
-   Exemplo de tuplos:
    -   `<"token">`, `<"moeda", "luis", 1€>`, `<"moeda", "luis", 2€>`
-   3 operações principais:
    -   🟢 **put** (≡ `out`): insere um tuplo
    -   🔵 **read** (≡ `rd`): lê um tuplo (sem o remover)
    -   🔴 **take** (≡ `in`): remove um tuplo (bloqueante, se não existir)

### 🃏 Wildcards

-   `take(<"moeda", "luis", *>)` pode retirar qualquer tuplo que **corresponda ao padrão**, logo tanto pode retirar `<"moeda", "luis", 1€>` como `<"moeda", "luis", 2€>`

---

## 🔒 Exclusão Mútua com Take

-   O `take` é **bloqueante**, logo pode ser usado para **sincronização entre processos**
-   Exemplo: usar o tuplo `<token>` como "chave"
    -   Processo faz `take(<token>)`, entra na secção crítica
    -   Quando termina, faz `put(<token>)`

➡️ Permite **partilha de memória e sincronização** de forma elegante e uniforme

---

## 🔁 Diferenças para registos

-   Podem existir **várias cópias do mesmo tuplo**
-   Escrita ≈ `take` + `put` (em vez de simples substituição)
-   `take` permite operações que exigiriam **instruções atómicas** em memória partilhada (ex: compare-and-swap)

---

## 🌐 Espaço de Tuplos Distribuído

-   É implementado usando **algoritmos de replicação**
-   Mantém tuplos **replicados em várias réplicas**
-   Objetivo: garantir **linearizabilidade**

---

## ⚙️ Considerações sobre a implementação

### 📦 Observação I: Filiação do grupo

-   É necessário um **serviço de filiação** para manter a lista de réplicas ativas
-   Quando há falhas, a filiação muda
-   Maiorias e confirmações são feitas com base na **filiação atual**

### 🔁 Observação II: Comunicação

-   Usa-se **UDP**
    -   O algoritmo trata da **retransmissão**
    -   Modularmente, podia usar-se TCP

---

## 🎯 Objetivos do algoritmo

-   Otimizar para que o cliente receba a resposta **o mais cedo possível**
-   Garantir **linearizabilidade**, mesmo com otimizações

---

# 📘 Algoritmo Xu-Liskov

> Retirado do livro da cadeira (onde aparece como "write", mas aqui é **put**)

### 📌 Operações

-   **put**:
    -   Cliente envia pedido a todas as réplicas
    -   Espera confirmação de todas
    -   Réplicas armazenam o tuplo
-   **read**:
    -   Cliente envia pedido a todas as réplicas
    -   Aguarda resposta da **primeira réplica com o tuplo**
    -   Se for com wildcard, pode precisar de mais verificação

---

## 🧩 Implementação em gRPC

-   ❌ Não se deve usar **blocking stubs**
-   ✅ Usar **stubs assíncronos** (non-blocking) → permite avançar com o código sem esperar

---

## 🔄 Relação com ABD

-   É uma **variação simplificada** do algoritmo ABD:
    -   Escrita: quorum = **todos**
    -   Leitura: quorum = **um**
-   ✅ Não precisa de versão, pois tuplos são únicos (diferente dos registos)

---

## 🛑 Problemas com read não bloqueante

-   Se `read()` devolvesse `null` quando não encontra o tuplo:
    -   Poderia **violar linearizabilidade**
    -   Exemplo:
        -   Cliente faz `put(t)`
        -   Outro cliente faz `read(t)` várias vezes em paralelo
        -   Pode ver `t` numa leitura, e depois ver `null`

🧠 Isto acontece porque:

-   O leitor pode receber respostas de **réplicas diferentes** em cada `read()`
-   Algumas réplicas já têm o tuplo `t`, outras **ainda não o aplicaram**
-   Resultado: uma leitura vê `t`, e a seguinte pode ver `null`

---

## 🧠 Discussão sobre Take

-   Proposta dos autores:
    -   **Solução customizada** que garante ordem total sobre o mesmo tuplo
    -   ⚠️ Não determinista: pode ser necessário **aleatoriedade**
    -   Pode falhar se nenhum processo conseguir maioria

### ✅ Alternativa:

-   Usar **protocolo de ordem total** → será estudado mais à frente

---

## ⚡ Otimizações para minimizar latência

### 🟢 Put otimizado:

-   Cliente pode **responder à app logo após enviar os pedidos**
-   Réplicas atualizam-se em segundo plano

### 🔴 Take otimizado:

-   No caso do `take`, o cliente pode responder à aplicação **assim que souber qual tuplo será retirado**
-   Quando o `take` usa **wildcards**, esta decisão pode ser feita **logo após a 1ª fase** da operação

⚠️ **Problema**: a réplica pode ainda não ter aplicado a operação → risco de incoerência

---

## ⚠️ Problemas de Coerência

### 🧪 Problema 1:

-   Cliente faz `take(<x>)` e depois `read(<x>)`
-   Réplica pode ainda não ter removido `<x>` → responde com `<x>`
-   Resultado: o cliente vê uma **história não linearizável**

✅ Solução:

-   Cada réplica deve **executar as operações do mesmo cliente na ordem correta**

---

### 🧪 Problema 2:

Exemplo com 2 réplicas:

-   `Worker1` faz `put(A)`, `take(A)`, `put(B)`
-   `Worker2` faz `read(B)`, depois `read(A)`

➡️ Pode acabar por ver **`A` depois de `B`**, mesmo que `A` tenha sido removido antes — **não linearizável**

---

## 🛡️ Prevenir a anomalia

-   O problema ocorre porque:
    -   `put` executa antes do `take` concluir em algumas réplicas
-   ✅ Solução:
    -   Quando um cliente faz `put`, **espera que a operação anterior (`take`) tenha terminado** (fase 2 concluída)
    -   Só depois envia `put` para as réplicas

---
