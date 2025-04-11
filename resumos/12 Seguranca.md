# 🔐 Segurança em Sistemas Distribuídos

---

## ⚠️ Vulnerabilidades e Ameaças

-   Sistemas têm **falhas técnicas** e erros humanos
-   Existem **diversos tipos de ataques**
-   A segurança requer:
    -   Conhecimento técnico
    -   Boas práticas de utilização
    -   Mecanismos específicos de defesa

---

## 📡 Comunicação Segura

### 👥 Participantes típicos:

-   **A** e **B** (Alice and Bob): entidades que comunicam
-   **T** (Trudy): espia e ataca
-   **Eve**: apenas espião
-   **Mallory**: atacante ativo

---

## 🔑 Mecanismos Básicos

-   **Cifra com chaves simétrica**
-   **Cifra com chaves assimétrica**
-   **Funções de hash**
-   **Nonces**

---

## 🔐 Cifra Simétrica

-   A e B partilham uma **chave secreta Ks**
-   A cifra a mensagem com `Ks(m)` e B decifra com a mesma chave
-   Exige **partilha prévia da chave** (ex: presencial, senha, canal seguro)

📌 A **segurança** depende:

-   Do tamanho da chave
-   E do **poder computacional do adversário**

➡️ Quanto **maior a chave**, **mais difícil é quebrá-la** por força bruta

### 🧮 Cifragem e Decifragem:

-   Para cifrar: `c = Ks(m)`
-   Para decifrar: `m = Ks(c)`
-   Ou seja: `m = Ks(Ks(m))` ✅

### 🔄 Exemplos:

-   **DES** – _Data Encryption Standard_ (56 bits) → inseguro
-   **3DES** – _Triple Data Encryption Standard_ → utiliza 3 chaves consecutivas
-   **AES** – _Advanced Encryption Standard_ (128/192/256 bits) → seguro e atual

---

## 🗝️ Cifra Assimétrica

-   Muito mais lenta que a criptografia simétrica

-   Cada entidade possui um par de chaves:
    -   **Chave pública** `K⁺` → conhecida por todos
    -   **Chave privada** `K⁻` → mantida em segredo
-   O que for cifrado com uma só pode ser decifrado com a outra:
    -   `K⁺(K⁻(m)) = m`
    -   `K⁻(K⁺(m)) = m`
-   Não é possível derivar `K⁺` a partir de `K⁻`, nem `K⁻` a partir de `K⁺`

### 📦 Usos típicos:

#### 🔒 Confidencialidade (enviar mensagem segura para B):

-   A cifra `m` com a **chave pública de B**: `K⁺_B(m)`
-   Apenas B pode decifrar com a sua **chave privada** `K⁻_B`

#### ✍️ Autenticação / Assinatura Digital:

-   A cifra `m` com a sua **chave privada** `K⁻_A`: `K⁻_A(m)`
-   B verifica a assinatura com a **chave pública de A**: `K⁺_A(K⁻_A(m)) = m`
-   Se a verificação for bem-sucedida, a mensagem foi de facto produzida por A

### 📌 Exemplos:

-   **RSA** (Rivest-Shamir-Adleman) – usado para cifragem e assinatura
-   **Diffie-Hellman** – usado para troca segura de chaves

---

## 🔀 Cifra Mista

-   Combina **cifra assimétrica** e **cifra simétrica**:
    -   A **cifra assimétrica** é usada **durante a criação do canal seguro**, para **negociar uma chave simétrica**
    -   A **cifra simétrica** é usada depois, para a **comunicação contínua** (mais eficiente)

### 🔑 Como funciona (fase de criação do canal seguro):

1. A (cliente) **gera uma chave simétrica aleatória `Ks`**
2. A **cifra `Ks` com a chave pública de B**: `K⁺_B(Ks)`
3. A envia `K⁺_B(Ks)` para B
4. B **decifra com a sua chave privada `K⁻_B`** e obtém `Ks`
5. A e B passam a comunicar através de **cifra simétrica com `Ks`**

### 📌 Vantagens:

-   Junta o **melhor dos dois mundos**:
    -   A cifra assimétrica garante **confidencialidade na troca da chave**
    -   A cifra simétrica permite **comunicação rápida e eficiente**

✅ Técnica usada em protocolos como **TLS/SSL** (ex: HTTPS)

---

## 🧮 Funções de Hash Criptográficas

-   Geram um **resumo fixo (digest)** de uma mensagem: `H(m)`
-   Usadas para **verificar integridade** e em **assinaturas digitais**
-   **Impossível inverter ou gerar colisões facilmente**

📌 Propriedades:

-   Digest tem **tamanho fixo**, independentemente de `m`
-   **Não é possível inverter** nem encontrar `m'` tal que `H(m) = H(m')`

### 🔒 Exemplos:

-   **MD5** → inseguro
-   **SHA-1** → inseguro e obsoleto
-   **SHA-2 / SHA-3** → seguros

---

## 🔄 Nonce

-   Valor único usado para **identificar uma troca de mensagens**
-   Nunca é reutilizado em comunicações futuras
-   Evita **ataques de repetição (replay attacks)**: o atacante não consegue reutilizar mensagens cifradas

📌 Mesmo que um intruso intercepte mensagens cifradas, **não pode reaproveitá-las**, pois o nonce muda em cada sessão

### 🔢 Tipos comuns:

-   **Timestamps**
    -   Requer relógios sincronizados
-   **Números de sequência crescentes**
    -   Requer que cada entidade memorize o último número usado

---

## 🧩 Combinação dos Mecanismos

-   Permite:
    -   Assinaturas digitais
    -   Infra-estruturas de chaves públicas e certificados
    -   Email seguro
    -   Autenticação (ex: `SSO` -> Single Sign-On)
    -   Canais seguros

---

## ✍️ Assinaturas Digitais

### 🔐 Com chave simétrica (MAC)

-   A e B partilham um segredo `Ks`
-   A calcula `Sm = H(m|Ks)` e envia `m + Sm`
-   B verifica se `Sm == H(m|Ks)`
-   ⚠️ Só funciona entre entidades que **partilham uma chave**
-   Designado por **Message Authentication Code (MAC)**

### 🗝️ Com chave assimétrica

-   A quer assinar uma mensagem `m`
-   Gera `H(m)` e cifra com a sua chave privada: `K⁻_A(H(m))`
-   Envia `m + K⁻_A(H(m))`
-   Qualquer entidade pode validar a assinatura com `K⁺_A`

📌 Garante:

-   **Autenticidade** da origem
-   **Integridade** da mensagem

---

## 🧨 Man-in-the-Middle Attack

-   O atacante **intercepta** e **modifica** a comunicação entre A e B
-   Acreditam que estão a comunicar diretamente

### 🛡️ Como evitar:

-   Se A já conhece `K⁺_B`
-   Uso de **certificados digitais**

---

## 🧾 Infraestrutura de Chave Pública (PKI)

-   **Certificados X.509** associam chave pública a uma entidade
-   São assinados por **Autoridades de Certificação (CA)**
-   Qualquer entidade pode validar um certificado, se confiar na CA

---

## 📬 Email Seguro com `PGP` (Pretty Good Privacy)

-   Cada utilizador possui um par de chaves assimétricas (K⁺, K⁻)
-   A chave pública é divulgada:
    -   Diretamente (ex: entrega em mão, página pessoal)
    -   Ou através de outros utilizadores que a **certificam** com as suas próprias chaves
    -   Versões recentes também suportam **infraestruturas de chave pública (PKI)**

### ✉️ Como enviar um email seguro:

1. O remetente **gera uma chave simétrica `Ks`**
2. Cifra a mensagem com `Ks`
3. Cifra `Ks` com a **chave pública do destinatário `K⁺_B`**
4. Envia:
    - A mensagem cifrada
    - A chave simétrica cifrada: `K⁺_B(Ks)`

### ✅ Garantias de segurança:

-   **Confidencialidade**: só o destinatário consegue obter `Ks` com `K⁻_B`
-   **Integridade e autenticidade**:
    -   O remetente gera uma **assinatura digital** da mensagem
    -   O destinatário valida com a **chave pública do remetente**

---

## 🌐 Canais Seguros com `TLS/SSL` (Transport Layer Security / Secure Sockets Layer)

-   Protocolo amplamente usado em HTTPS e suportado por quase todos os browsers e servidores
-   Permite criar **canais seguros** sobre TCP
-   Disponível via interface `secure socket` (ex: bibliotecas em C, Java...)

### 🔐 Garante:

-   **Confidencialidade**
-   **Integridade**
-   **Autenticação**

### 🔧 Criação de Canal Seguro (versão simplificada):

1. A (ex: browser) pede a `K⁺_B` (pública do servidor) e envia `nonce`
2. B responde com:
    - Certificado com `K⁺_B`
    - Outro `nonce`
3. A verifica o certificado. Se inválido, **aborta**
4. A gera um **segredo (`master secret`)** usando os `nonces`
5. A envia o `master secret` cifrado com `K⁺_B`
6. A e B derivam **4 chaves simétricas** a partir do `master secret`:
    - `Kc` → cifra cliente → servidor
    - `Mc` → MAC cliente → servidor
    - `Ks` → cifra servidor → cliente
    - `Ms` → MAC servidor → cliente

### 📦 Comunicação com Records:

-   Dados enviados em blocos chamados **records**
-   Cada record:
    -   É **assinado com MAC**: `MAC = Hash(record | M | tipo | nº sequência)`
    -   É **cifrado** para garantir a confidencialidade
-   Impede alterações, reordenação ou repetições sem serem detetadas

✅ A utilização de múltiplas chaves aumenta a robustez da ligação

---

## 👤 Autenticação do Cliente

-   TLS autentica **o servidor**, mas **não o cliente**
-   Em muitos casos, o servidor precisa de saber **quem é o cliente** (ex: acesso restrito)

### 🛠️ Soluções possíveis:

-   📄 **Certificados com chaves públicas**

    -   Pouco prático: o cliente teria de manter e proteger o certificado em todos os dispositivos

-   🔑 **Palavra-passe partilhada**
    -   Simples, mas inseguro se usado em muitos serviços
    -   Problemas:
        -   Senhas fracas ou repetidas
        -   Escritas em papel ou visíveis
        -   Usam as mesmas em várias aplicações

---

## 🔐 Single Sign-On (SSO)

-   Cliente autentica-se **uma vez apenas** num **servidor de autenticação**
-   Outros serviços confiam nesse servidor
-   ✅ O cliente:
    -   Só partilha segredos com o SSO
-   ✅ Os serviços:
    -   Só precisam de confiar no SSO, não em cada cliente

📌 Base para serviços como **OAuth**, **Google Login**, etc.

---

## 🧾 Protocolo de Needham-Schroeder

-   Protocolo clássico de autenticação usando **chaves simétricas**
-   O cliente obtém um **ticket** de sessão para se autenticar junto dos serviços
-   Base para o sistema **Kerberos**

⚠️ A versão original é vulnerável a **ataques de repetição (replay attacks)**

-   Pode ser melhorado com **timestamps** para garantir validade

---

## 🔐 SSO com chaves assimétricas

-   O mesmo princípio pode ser usado com **chaves públicas/privadas**
-   Exemplo: login com Google/Facebook → autenticação central com chave pública

---
