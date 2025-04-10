# 🧬 Replicação e Registos Replicados

## 🔄 Breve introdução à replicação

### 📦 Replicação

-   Manter **cópias dos dados e software** do serviço em **vários computadores**
-   Os **clientes** interagem com **gestores de réplicas**, que tratam da coordenação
-   Possibilita distribuir carga e **melhorar desempenho e disponibilidade**

---

## 🎯 Benefícios da replicação

-   ⚡ **Desempenho e escalabilidade**

    -   Clientes podem aceder à **réplica mais próxima**
    -   Carga distribuída → maior escalabilidade

-   ✅ **Disponibilidade**
    -   Sistema pode continuar funcional mesmo com **falhas de nós** ou da **rede**

---

## 🧠 Coerência

-   O objetivo é garantir que as leituras devolvem o **valor mais recente**, mesmo que a escrita tenha sido feita noutra réplica
-   Critério forte: **Linearizabilidade**

---

## 🧭 Linearizabilidade

Um sistema é **linearizável** se:

1. Existe uma ordem global das operações que respeita a ordem real (tempo)
    - Se `op1` termina antes de `op2` começar, então `op1` deve estar antes de `op2` na serialização
    - Se forem concorrentes, a ordem pode ser arbitrária
2. Cada cliente observa resultados **coerentes com essa ordem global**

---

## 📋 Ordenar operações

-   Operações não são instantâneas (pedido → execução → resposta)
-   Exemplo:
    -   Se `X` começa depois de `Y` terminar → `X` ocorre depois de `Y`
    -   Se `X` começa antes de `Y` terminar → `X` é **concorrente** com `Y`

---

## 🧪 Exemplos de linearizabilidade

Verificar `04 Replicacao Registos replicados.pdf` - Páginas 9, 10 e 11

-   Exemplo I: linearizável ✔️
-   Exemplo II: verificar → ❌
-   Exemplo III: verificar → ❓

---

## 🧰 Registos Partilhados (Replicados)

-   Objetivo: simular **um registo partilhado**
-   Duas operações principais:
    -   📝 `write(v)`
    -   🔍 `read()`

### 🧾 Regras:

-   Uma nova escrita **substitui** a anterior
-   Apenas **um cliente pode escrever**
    -   Escritas são **totalmente ordenadas**
-   Vários clientes podem ler

---

## 📏 3 Modelos de Coerência para Registos (Lamport)

-   **Atomic** (≡ linearizável)
-   **Regular**
-   **Safe** (não estudado na cadeira)

---

## ✅ Registo Atomic

-   Equivalente a **linearizabilidade aplicada a registos**
-   Leituras e escritas ocorrem como se fossem **instantâneas**
-   O valor devolvido numa leitura corresponde a um ponto entre o início e fim da operação

---

## ⚠️ Registo Regular

-   Se leitura **não for concorrente** com escrita → devolve último valor escrito
-   Se for **concorrente** com uma escrita → pode devolver o valor anterior **ou** o novo
-   ❌ Permite resultados incoerentes em certas interleavings

🧠 **Exemplo de incoerência**:

> Enquanto uma escrita está a decorrer, duas leituras seguidas (uma a seguir à outra) podem devolver uma sequência **incoerente de valores** — por exemplo:
>
> -   1ª leitura: devolve o **novo valor**
> -   2ª leitura: devolve o **valor antigo**

➡️ Isto viola a intuição de que o tempo avança e os dados devem refletir essa progressão.

---

## 📌 Implementar registos distribuídos

-   Cada processo mantém uma cópia com `<valor, versão>`
-   Operações são feitas por troca de mensagens entre processos
-   Possível implementar com:
    -   Tolerância a falhas
    -   Sem bloqueios (não impede leitura durante escrita)

---

# 🧮 Algoritmo ABD (Attiya, Bar-Noy, Dolev)

## ✍️ Escrita (com um único escritor)

1. Escritor incrementa versão e envia `<valor, versão>` para todos
2. Cada processo:
    - Atualiza a cópia **se versão for superior**
    - Envia ACK
3. Escrita conclui quando há ACK de **uma maioria**

---

## 📖 Leitura (com qualquer leitor)

1. Leitor envia pedido a todos os processos
2. Cada processo responde com o seu `<valor, versão>`
3. Leitor:
    - Escolhe o valor com a **maior versão**
    - Atualiza o seu próprio valor, se necessário
    - Retorna o valor

---

## 💡 Intuição do algoritmo

-   Maioria escrita + maioria lida → **intersecção garantida**
-   Garante que se a leitura vem após uma escrita concluída, o valor mais recente será lido

---

## 🧪 Exemplos

### Exemplo 1:

-   Escrita <v1, t1> chega a 2 réplicas
-   Leitura 1 lê de réplica desatualizada → devolve v0
-   Leitura 2 lê da maioria → devolve v1

### Exemplo 2:

-   Escrita enviada mas mensagem atrasa
-   Leitura pode ocorrer antes da escrita ser completada
-   Verificar se o resultado é compatível com atomicidade

---

## 🧠 De Regular → Atomic

-   É possível adaptar o algoritmo para garantir **atomicidade**, de forma **não bloqueante**

### Leitura atomic:

1. Faz a leitura (como antes), obtendo valor mais recente
2. **Executa uma "escrita fictícia"** com o mesmo valor (reforça a ordem)
3. Só depois devolve o valor ao cliente

➡️ Isto garante que futuras leituras verão o mesmo valor ou mais recente

---
