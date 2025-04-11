# 🔒 Exclusão Mútua em Sistemas Distribuídos

## 🧠 Motivação

-   O problema é semelhante ao dos sistemas operativos: garantir que **dois processos não acedem ao mesmo recurso crítico ao mesmo tempo** (ex: ficheiros, estruturas de dados, dispositivos).
-   Em sistemas distribuídos, a ausência de memória partilhada torna esta coordenação mais desafiante.

## 🛠️ Solução: Exclusão Mútua Distribuída

-   Os processos devem **coordenar-se entre si** para garantir acesso exclusivo ao recurso.
-   A exclusão mútua é alcançada **exclusivamente através da troca de mensagens** — não há memória partilhada nem coordenador físico comum.

---

## 🧩 Solução Centralizada (Cliente-Servidor)

### Como funciona:

-   Um processo atua como **servidor** que guarda a "chave" de acesso ao recurso.
-   Um cliente que quer aceder envia pedido ao servidor.
-   O servidor:
    -   Entrega a chave se estiver disponível
    -   Caso contrário, coloca o cliente numa **lista de espera**
-   Quando a chave é devolvida, é passada ao próximo da fila.

### ❌ Limitações:

-   Ponto único de falha (servidor)
-   Sobrecarga do servidor
-   A chave tem sempre de passar pelo coordenador

➡️ Podia ser mais eficiente se o cliente pudesse **passar a chave diretamente** ao próximo ➡️ ideia para descentralização.

---

## 🔁 Algoritmo de Ricart & Agrawala

### ✅ Características:

-   Algoritmo **descentralizado**
-   Usa apenas **troca de mensagens**
-   Cada processo comunica com **todos os outros**
-   Permite que um cliente envie diretamente a "chave" para o próximo

### ❌ Desvantagens:

-   **Não é tolerante a falhas**
-   Todos os processos ficam sobrecarregados (sobrecarga distribuída)
-   Exige **relógios lógicos** (ex: Lamport) para desempate de prioridades
-   Pode ser adaptado para usar **prioridade fixa** (ex: ID do processo)

➡️ É um passo interessante para soluções totalmente distribuídas.

---

## 🔗 Formalização com relógios lógicos

-   Cada pedido inclui timestamp (relógio lógico)
-   Um processo só entra na secção crítica após receber **resposta de todos os outros processos** com timestamp maior
-   Se dois pedidos tiverem o mesmo timestamp, desempate pode ser feito com o ID do processo

---

## 🧮 Algoritmo de Maekawa

### ✅ Características:

-   **Reduz carga** em comparação com Ricart & Agrawala
-   Os processos estão organizados em **quóruns** (subconjuntos)
-   Cada processo pode pertencer a **vários quóruns**
-   Um processo precisa de **votos de todos os processos do seu quórum** para entrar na secção crítica

### 📌 Regras dos quóruns:

-   Qualquer par de quóruns tem interseção não vazia: `Vi ∩ Vj ≠ ∅`
-   Um processo **só pode votar num pedido de cada vez**

### ❌ Limitações:

-   Pode ocorrer **interbloqueio** (deadlock)
-   Requer mecanismos de deteção e recuperação de bloqueio

➡️ Melhor escalabilidade, mas precisa de mais complexidade na gestão.

---

## 🧠 Comparação entre Soluções

### 📈 Eficiência (mensagens, atrasos)

| Algoritmo         | Mensagens (entrada/saída) | Atraso de cliente | Atraso de sincronização |
| ----------------- | ------------------------- | ----------------- | ----------------------- |
| Centralizado      | 3                         | 2                 | 2                       |
| Ricart & Agrawala | 2 × (N - 1)               | 2                 | 1                       |
| Maekawa           | 3 × quorum_size           | 2                 | 2 📌                    |

📌 Supõe-se que dois quóruns se intersectam em apenas 1 processo.

---

### ⚖️ Distribuição da carga

-   **Centralizado**: tudo passa pelo coordenador → pode sobrecarregar
-   **Ricart & Agrawala**: todos os processos recebem pedidos → sobrecarga distribuída
-   **Maekawa**: apenas os processos do quórum são afetados → melhor escalabilidade

---

### ⚠️ Tolerância a falhas

| Algoritmo         | Tolerância                                                                                                            |
| ----------------- | --------------------------------------------------------------------------------------------------------------------- |
| Centralizado      | ❌ Falha do servidor bloqueia o sistema<br>✅ Tolera falha de clientes que **não detenham nem tenham pedido a chave** |
| Ricart & Agrawala | ❌ Não tolera a falha de nenhum processo                                                                              |
| Maekawa           | ✅ Tolera falha de processos **fora do quórum**<br>❌ Falha dentro do quórum pode bloquear acesso                     |

📌 Nenhuma das soluções tolera **perdas de mensagens** → assumem **canal fiável**

---
