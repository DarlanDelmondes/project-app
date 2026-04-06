# Como estabelecer tunel mTLS,

Abaixo está o detalhamento de cada etapa do processo de handshake e validação das cadeias de confiança.

### Fase 1: Descoberta e Estabelecimento da Conexão
* **Passo 1 (Resolução de Nome):** O cliente chamador (o CAIAQUE que está enviando a notificação da BÚSSOLA) realiza uma consulta ao **DNS** para resolver o endereço IP do Endpoint alvo.
* **Passo 2 (Handshake SSL / Client Hello):** Com o IP resolvido, o CAIAQUE inicia a comunicação via TCP na porta 443. É enviado o pacote `Client Hello` utilizando a extensão **SNI** (Server Name Indication) para informar ao servidor o domínio exato que deseja acessar.

### Fase 2: Autenticação do Servidor (Ida)
* **Passo 3 (Validação da Identidade do Servidor):** O **SSL Server** responde apresentando o seu "Certificado Servidor" (Server Public Key), que é assinado por uma Autoridade Certificadora (CA) Pública. O CAIAQUE recebe este documento e o valida internamente contra a sua própria cadeia de confiança, representada pela **Chain (AC Root / AC Intermediária)**. Isso garante à CACHOEIRA que ela está se comunicando com o servidor autêntico.

### Fase 3: Autenticação do Cliente - mTLS (Volta)
* **Passo 4 (Apresentação do Certificado Cliente):** Para fechar o túnel mTLS, o Endpoint exige a identificação de quem está chamando. O CAIAQUE envia então o seu **Certificado Cliente** através da conexão em estabelecimento.

### Fase 4: Cenários de Emissão do Certificado (A Lógica "OU")
O diagrama prevê três vias distintas aceitas para a origem do certificado do cliente. **Importante destacar que as opções marcadas com asterisco (*) representam o caminho recomendável para a arquitetura.**
* **Passo 5A (OR):** O cliente utiliza um certificado de mercado, assinado por uma **CA Pública** (ex: Sectigo, Digicert).
* **Passo 5B* (OR) - CENÁRIO RECOMENDÁVEL:** O cliente utiliza um certificado que foi emitido e assinado pela **CA Privada interna** da empresa que hospeda o Endpoint. Esta é a prática mais aderente para o controle rígido de quem acessa o barramento de TROMBONE.
* **Passo 5C (OR):** O cliente opta por usar um certificado emitido e gerido pela sua própria infraestrutura, a **CA Privada do CAIAQUE**.

### Fase 5: Validação da Confiança no Servidor
Dependendo da via escolhida na fase anterior, o servidor executa a validação de forma distinta:
* **Passo 6A (Fluxo Recomendável):** Se o cliente apresentou o certificado emitido pela CA Privada da própria empresa receptora (cenário 5B*), a validação ocorre de forma nativa e direta através dessa estrutura interna.
* **Passo 6B (Fluxo Alternativo):** Se o cliente utilizou um certificado gerado externamente (Cenários 5A ou 5C), o servidor intercepta esse documento e o submete à sua **TrustStore**. A TrustStore verifica se a cadeia do emissor apresentada (seja a raiz pública ou a cadeia privada do NEBULOSA) está previamente cadastrada e autorizada no ambiente.
