# 🆕 Introdução a SD

## 💡 O que é um Sistema Distribuído?

Um **sistema distribuído** é formado por vários processos, tipicamente localizados em diferentes máquinas, que cooperam para realizar uma tarefa comum.

Esta cooperação é feita através da **troca de mensagens**.

---

## 🛠️ Programar um Sistema Distribuído

### ⚙️ Desafios principais

Para programar um SD é necessário:

-   📐 Definir uma arquitetura adequada (ex: cliente-servidor)
-   🧠 Escolher um modelo de programação
-   🛡️ Gerir falhas, concorrência e comunicação

### 🤝 Arquitetura Cliente-Servidor

Neste modelo, os **clientes fazem pedidos** e o **servidor responde**.

É uma das formas mais simples e comuns de estruturar uma aplicação distribuída.

### 🧪 Projeto hipotético de exemplo

Implementar um **servidor de contagem** com três operações:

-   🧼 `Limpa`: reinicia o contador
-   ➕ `Incrementa`: adiciona um valor ao contador
-   🔍 `Consulta`: devolve o valor atual

Este projeto assume:

-   🌐 Uma **rede não fiável** (mensagens podem perder-se ou chegar fora de ordem)
-   🖥️ Um **sistema heterogéneo** (representações de inteiros podem variar entre máquinas)

---

## 🔄 Comunicação entre processos

### 🔁 Modelo de comunicação Request-Reply

O cliente envia um **pedido**, o servidor **processa** e devolve uma **resposta**.  
É um modelo direto e comum em SDs simples.

---

### 🌐 TCP ou UDP?

Escolher o protocolo de transporte certo é essencial.

#### ✅ TCP

-   Garante **entrega fiável** e ordenada.
-   Usa _handshake_ para estabelecer ligação: **SYN → SYN-ACK → ACK**.
-   Tem controlo de fluxo e deteção de perdas.

🛑 Desvantagens:

-   Cada invocação implica **mensagens extra** (`SYN`, `ACK`, `FIN`).
-   A sobrecarga é elevada para **operações simples**.
-   A **resposta** já pode servir de confirmação → **ACK torna-se redundante**.

#### 🚀 UDP

-   Protocolo **leve**, **sem ligação**, e com **menos overhead**.
-   **Não garante entrega nem ordem**, mas é aceitável se o cliente puder lidar com isso.

🧠 Neste projeto (operações simples e curtas), **assume-se o uso de UDP**.

---

### 🧾 Estrutura das mensagens

Cada mensagem inclui:

-   🆔 **Identificador do pedido** (cliente + número sequencial)
-   🔢 **Operação** (`Limpa = 0`, `Incrementa = 1`, `Consulta = 2`)
-   📦 **Argumentos ou retorno**, serializados

---

## 📦 Serialização e Marshalling

### 🎁 Necessidade de serializar

Antes de enviar dados pela rede, é necessário transformá-los numa **sequência de bytes**.

### 🌍 Problemas de heterogeneidade

Máquinas distintas podem representar dados (como inteiros) de formas diferentes.

### 🔄 Marshalling e Unmarshalling

-   **Marshalling**: conversão + normalização para formato comum
-   **Unmarshalling**: reconstrução local a partir dos bytes recebidos

---

## ⚠️ Gestão de falhas

### 🧨 Possíveis falhas com UDP

-   🕳️ Perda de mensagens
-   📤 Repetição de mensagens
-   ⏳ Chegada fora de ordem
-   💥 Crash de processos (falhas silenciosas)

---

### ⏱️ Timeout no cliente e reenvio

Se o cliente **não recebe resposta** dentro do tempo limite:

-   ❌ Pode **retornar erro**
-   🔁 Ou **reenviar o pedido**, várias vezes se necessário

#### ⚠️ Problema: Reenvio pode causar duplicação

-   A **resposta do envio original pode ainda chegar**
-   O servidor **não reconhece** o pedido como duplicado
-   Pode **executar a operação 2 vezes**

---

#### 🧠 Exemplo de cenário

1. ⏱️ `T0`: cliente envia pedido
2. 📭 A resposta atrasa-se
3. 🔁 Cliente reenviou
4. ✅ Resposta de qualquer um pode chegar
5. 💣 O servidor executa **duas vezes**

➡️ O servidor não distingue entre o original e o reenvio e pode executar a operação duas vezes.

---

#### 💡 O que é uma operação idempotente?

Uma operação é **idempotente** se, mesmo executada várias vezes com os mesmos dados:

-   Produz o **mesmo estado final** no servidor.
-   Retorna a **mesma resposta** ao cliente.

Exemplos:

-   ✅ `Limpa`: **É idempotente** (definir o contador a 0 repetidamente não muda o resultado).
-   ⚠️ `Incrementa`: **NÃO é idempotente** (cada chamada aumenta o valor do contador).

---

#### 🛡️ Como evitar execuções repetidas?

Para evitar este problema, o **servidor deve guardar estado** associado a cada pedido.

-   Verifica o `id_pedido`:
    -   🆕 Se for **novo** → executa a operação e guarda a resposta.
    -   ♻️ Se for **repetido** → **não executa novamente**, devolve a **resposta previamente armazenada**.

##### 📁 Pode armazenar algo como:

(id_cliente, número_sequencial, resposta)

---

#### 📈 Questões de escalabilidade

-   ⏳ Durante quanto tempo guardar os pedidos?
-   📊 Quantos pedidos por cliente?
-   🚀 Como suportar milhares de clientes sem sobrecarregar memória?

---

## 🧰 Abstração com Remote Procedure Call (RPC)

### 🔗 O que é RPC

Permite que o programador **chame funções remotamente** como se fossem locais.

### 📡 gRPC

-   Tecnologia moderna de RPC
-   Usada no projeto
-   Interfaces definidas com ficheiros `.proto`

### 🧑‍💻 Exemplo com gRPC

-   O cliente usa um **stub**
-   O servidor implementa métodos como `incrementa()`
-   O protocolo trata da comunicação e (de)serialização

### 🧮 Semânticas de Invocação em RPC

Dependendo das **medidas de tolerância a falhas** implementadas, uma chamada RPC pode ter diferentes **semânticas de invocação** (ou "call semantics"). Estas semânticas definem o **comportamento esperado** do sistema em caso de falhas.

| 📤 Reenvio do pedido | 🧹 Filtragem de duplicados | 🔁 Reexecução ou retransmissão | 📞 Semântica da chamada |
| -------------------- | -------------------------- | ------------------------------ | ----------------------- |
| ❌ Não               | 🚫 Não aplicável           | 🚫 Não aplicável               | 🟡 _Maybe_ (talvez)     |
| ✅ Sim               | ❌ Não                     | ✅ Reexecuta procedimento      | 🔁 _At-least-once_      |
| ✅ Sim               | ✅ Sim                     | 📬 Retransmite resposta        | ✅ _At-most-once_       |

#### ✔️ Explicação das semânticas:

-   **Maybe**: o pedido pode ou não ser executado, e o cliente pode não saber o resultado. Nenhuma garantia é oferecida.
-   **At-least-once**: o pedido é garantidamente executado **uma ou mais vezes** — útil para operações idempotentes.
-   **At-most-once**: o pedido é executado **no máximo uma vez**, mesmo com reenvios — garante **sem duplicações**.

💡 **Nota:** A semântica _At-most-once_ é a mais segura, mas exige mais recursos (armazenamento de histórico de pedidos + filtragem).

---

## 🔁 Troca de mensagens

### 📬 Comunicação baseada em mensagens

-   RPC (chamadas remotas)
-   Difusão em grupo
-   Publicação/subscrição de eventos

### 🧠 Memória partilhada distribuída

-   Acesso comum a dados através de **espaços de tuplos**
-   Permite colaboração sem troca explícita de mensagens

---

## 🚧 Desafios dos sistemas distribuídos

### 🔀 Concorrência

-   Vários processos executam ao mesmo tempo
-   Necessário garantir **sincronização e consistência**

### 🕒 Latência

-   Distâncias físicas + limites da luz = atrasos inevitáveis
-   Coordenação entre continentes é sempre **lenta**

### 🔄 Coerência

-   Alterações de estado não são instantâneas para todos
-   Precisamos de definir **modelos de consistência**

### 🔁 Tolerância a falhas

-   Faltas de rede (perda de mensagens, reordenação de mensagens, partições de rede), processos, partições, comportamentos bizantinos
-   O sistema deve **continuar funcional** apesar disso

### 🔍 Deteção de falhas

-   Difícil distinguir entre:
    -   Falha real
    -   Atraso
    -   Problemas na rede

### 📊 Escalabilidade

-   O sistema deve crescer em:
    -   Número de utilizadores
    -   Volume de dados
    -   Nós na rede

...sem perder **desempenho**

### 🔐 Segurança

-   Mais processos = maior superfície de ataque
-   Deve garantir:
    -   Autenticidade
    -   Confidencialidade
    -   Integridade

---

## 🌍 Exemplo de sistema distribuído: DNS

### 🧭 O que é o DNS

Sistema que **resolve nomes** (ex: `www.google.com`) para **endereços IP**.  
É um exemplo clássico de sistema distribuído global.

### ⚙️ Funcionamento

-   Estrutura hierárquica de servidores:
    -   `root`
    -   `.pt`, `.com`, etc.
    -   Servidores autoritativos

#### Modos de resolução

-   🔁 **Iterativa**: cliente pergunta a cada servidor
-   🌀 **Recursiva**: um servidor resolve tudo

### 📥 Uso de cache

-   Cada organização pode ter cache local
-   ⚠️ Pode gerar problemas de coerência (dados desatualizados)

### 📈 Estratégias de escalabilidade

-   Replicação e geo-replicação de servidores
-   Hierarquia bem definida
-   Distribuição de carga e tolerância a falhas

---
