---
layout: manual
title: Manual de integração Cartão Protegido
description: Integração técnica Tokenização via REST API Braspag
search: true
translated: true
categories: manual
tags:
  - Cartão Protegido
language_tabs:
  json: JSON
  shell: cURL
---

# Introdução ao Cartão Protegido

A plataforma do **CARTÃO PROTEGIDO** é uma armazenadora segura de cartões de crédito. Os dados nela armazenados seguem as normas PCI, que garante a integridade das informações dos cartões armazenados.
<br><br>
O gateway PAGADOR (BRASPAG) está integrado ao **CARTÃO PROTEGIDO**, facilitando o envio e processamento de transações de cartão de crédito via token.

## Sobre o Produto

Enquanto produto, o CARTÃO PROTEGIDO, por ser uma solução para resolver o problema de armazenamento seguro de dados de cartão de crédito, pode ser utilizado para diversos fins, como:

* **Compra com 1 clique**: A “compra com 1 clique” permite que um pagamento online, via cartão de crédito, seja feito pulando a etapa de preenchimento dos dados para pagamento ou até mesmo de todo o processo do carrinho de compras, pois os dados do cartão já foram previamente informados pelo comprador em compras passadas e serão replicados em futuras compras mediante sua autorizaçãol.
* **Cobrança Recorrente**: Estabelecimentos que já possuam uma solução interna de gerenciamento de recorrências podem utilizar a plataforma apenas para a parte sensível: armazenar os dados de cartão de crédito e processar via PAGADOR, as cobranças nas Adquirentes utilizando apenas o token. (Não é obrigatório que a transação seja processada via Pagador)
* **Re-tentativa de envio de transação (venda)**: Para estabelecimentos que represam os dados da venda para passar num segundo momento, sendo para retentar o envio de uma transação de cartão de crédito para uma Adquirente ou para fazer algum procedimento interno antes de autorizar a venda (validação de estoque, análise de fraude), a plataforma atende perfeitamente esta finalidade. O estabelecimento precisa conhecer e manipular apenas um token, mantendo-se aderente com as regras de segurança da indústria de cartões de crédito.
* Ou para qualquer outra finalidade, onde se faça necessário armazenar dados de um cartão de crédito de forma segura, mesmo que temporariamente

## Sobre este manual

Este manual tem como objetivo orientar o desenvolvedor da loja sobre a integração com a plataforma **CARTÃO PROTEGIDO**, descrevendo as funcionalidades existentes e os métodos a serem utilizados, listando informações a serem enviadas e recebidas e provendo exemplos.

## Ambientes

### Ambiente Sandbox

Experimente as nossas APIs sem compromisso!

|Informação|Descrição|
|---|---|
|Credenciais de Acesso à API|Acesse o [Cadastro do Sandbox](https://cadastrosandbox.braspag.com.br/) e crie uma conta de testes.<BR>Ao fim do cadastro você receberá um `MerchantId` e um `MerchantKey`,<BR> que deverão ser utilizados para autenticar todas as requisições feitas para os endpoints da API|
|Endpoint Transacional|**https://apisandbox.braspag.com.br/**|
|Endpoint Cartão Protegido|**https://cartaoprotegidoapisandbox.braspag.com.br/**|

<aside class="warning">Para receber a MerchantID do Cartão Protegido, solicite à nossa equipe de implantação através da ferramenta [Suporte](https://suporte.braspag.com.br).</aside>
<aside class="warning">Para receber a URL de Produção, solicite à nossa equipe de implantação através da ferramenta [Suporte](https://suporte.braspag.com.br).</aside>

## Arquitetura

A integração é realizada através de serviços disponibilizados como Web Services. O modelo empregado é bastante simples: Através do endpoint https://cartaoprotegidoapisandbox.braspag.com.br/v1/ serão enviadas todas as requisições relativas à esse serviço. Essa URL recebera as mensagens HTTP através dos métodos POST, GET ou Del. Cada tipo de mensagem deve ser enviada para um endereço identificado através do "path".

* **POST** - O método HTTP POST é utilizado na criação do token.
* **DEL** - O método HTTP DEL é utilizado para remoção de token.
* **GET** - O método HTTP GET é utilizado para consultas de recursos já existentes. Por exemplo, consulta de tokens já criados.

## Sobre a Integração

Nas seções abaixo, estão graficamente representados, os fluxos do processo de venda. Existem 3 maneiras de integrar o produto:

* Diretamente pela plataforma do CARTÃO PROTEGIDO;
* Via plataforma PAGADOR, utilizando Webservice;
* Via plataforma PAGADOR, utilizando Post de Dados.

Os dados necessários para armazenar um cartão de crédito na plataforma são: CPF do Cliente, Nome do Cliente, Nome do Portador, Número do Cartão e Data de Validade. O código de segurança não é armazenado (vide seção Código de
Segurança).

<aside class="notice">A plataforma do CARTÃO PROTEGIDO armazena de forma segura, 100% PCI Compliance, os dados dos cartões de crédito.</aside>

<aside class="notice">Como a autorização de uma transação é via PAGADOR, todas as funcionalidades de confirmação da transação - Segundo Post (post de confirmação), e Terceiro Post (sonda) - permanecem funcionando da mesma forma. </aside>

# **PARÂMETRO** Alias

Este parâmetro tem por finalidade facilitar o armazenamento, por parte do cliente, de informações referentes a um Cartão Protegido. O cliente poderá, no momento do salvamento do cartão, criar um Alias (apelido) que identificará esse cartão na Plataforma CARTÃO PROTEGIDO. Outra vantagem, é o fato desse Alias poder ser associado a um novo token, o que facilitaria a troca de um cartão quando, por exemplo, a validade deste expirar. Para isso, o lojista deveria indicar que o token está desabilitado. Dessa forma, o Alias associado a ele ficaria liberado para ser utilizado com um novo token.

## Forma Correta de Associação

Um Alias pode ser associado a um novo Token, desde que antes seja desassociado do Token antigo, conforme indicado no exemplo abaixo:

|Merchant Id|Token|Alias|Enabled|
|-----------|--------------------|-----|-------|
|LOJA A|Token 1|XPTO|0|
|LOJA A|Token 2|XPTO|0|
|LOJA A|Token 3|XPTO|1|

<aside class="notice">Obs.: A desassociação ocorrerá após a exclusão do token criado. </aside>

## Forma de Associação Não Aceita

Um Alias pode ser associado a um novo Token, desde que antes seja desassociado do Token antigo, no exemplo abaixo está indicada uma forma que não permitiria essa associação, pois o Alias só estará liberado para uma nova associação
desde que esteja desvinculado de um determinado Token:

|Merchant Id|Token|Alias|Enabled|
|-----------|--------------------|-----|-------|
|LOJA A|Token 1|XPTO|1|
|LOJA A|Token 2|XPTO|1|
|LOJA A|Token 3|XPTO|1|

# Autenticação

A solução é composta pelo passo de solicitação de token de acesso via API. As funções, aqui apresentadas, precisam apresentar em seu header um token de acesso do tipo "BEARER" que expira e precisa ser solicitado novamente.

## Tokens de Acesso

O CARTÃO PROTEGIDO utiliza o protocolo padrão de mercado OAuth 2.0 para autorização de acesso a seus recursos específicos por ambientes, que são: **Sandbox** e **Produção**.

Esta sessão descreve o fluxo necessário para que aplicações cliente obtenham tokens de acesso válidos para uso na API.

## Obtenção do token de acesso  

O token de acesso é obtido através do fluxo oauth **client_credentials**. O diagrama abaixo, ilustra, em ordem cronológica, a comunicação que se dá entre a **Aplicação Cliente**, a **API BraspagAuth** e o **CARTÃO PROTEGIDO**.

1. A **Aplicação Cliente**, informa à API **BraspagAuth** sua credencial.

2. O **BraspagAuth** valida a credencial recebida. Se for válida, retorna o token de acesso para o **CARTÃO PROTEGIDO**.

3. A **Aplicação Cliente** informa o token de acesso no cabeçalho das requisições HTTP feitas ao **CARTÃO PROTEGIDO**.

4. Se o token de acesso for válido, a requisição é processada e os dados são retornados para a **Aplicação Cliente**.

<aside class="warning">Solicite à equipe de suporte o seu "client_id" e "client_scret" após concluir o desenvolvimento em sandbox. Os dados apresentados abaixo são apenas exemplos.</aside>

## Como obter o token

Uma vez em posse da credencial, será necessário "codificá-la" em Base64, utilizando a convenção **client_id:client_secret**, e enviar o resultado no cabeçalho através do campo **Authorization**.

Exemplo:
* client_id: **bb147542-4cbc-3957-a559-c73107e1f7d0**
* client_secret: **1q2w3e4r5t6y7u8i9o0p0q9w8e7r6t5y4u3i2o1p**
* String a ser codificada em Base64: **bb147542-4cbc-3957-a559-c73107e1f7d0:1q2w3e4r5t6y7u8i9o0p0q9w8e7r6t5y4u3i2o1p**
* Resultado após a codificação: **YnJhc3BhZ3Rlc3RlczoxcTJ3M2U0cg==**

<aside class="warning">Para o ambiente de homologação você deve utilizar o seguinte resultado dessa codificação:

"Basic ZTFjNTQ1NDItNGNiYy00OTU4LWI0NDktYjczMTA3ZTFmNmMwOmtibS9VRnRnMGZiaDhPSkVaRGJPUG1LU3ZSTElxK3RqeTh2UzZzNnppWEU9"</aside>

### Request

<aside class="request"><span class="method post">POST</span> <span class="endpoint">oauth2/token</span></aside>

**Parâmetros no cabeçalho (Header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/x-www-form-urlencoded|
|`Authorization`|Basic ZTFjNTQ1NDItNGNiYy00OTU4LWI0NDktYjczMTA3ZTFmNmMwOmtibS9VRnRnMGZiaDhPSkVaRGJPUG1LU3ZSTElxK3RqeTh2UzZzNnppWEU9|

**Parâmetros no corpo (Body)**

|Key|Value|
|:-|:-|
|`grant_type`|client_credentials|

### Response

``` json
{
  "access_token": "faSYkjfiod8ddJxFTU3vti_ ... _xD0i0jqcw",
  "token_type": "bearer",
  "expires_in": 599
}
```

**Parâmetros no corpo (Body)**

|Parâmetro|Descrição|
|:-|:-|
|`access_token`|O token de acesso solicitado. O aplicativo pode usar esse token para se autenticar no recurso protegido|
|`token_type`|Indica o valor do tipo de token|
|`expires_in`|Expiração do o token de acesso, em segundos <br/> O token quando expirar, é necessário obter um novo|


# SALVANDO O CARTÃO DURANTE A AUTORIZAÇÃO VIA PAGADOR

Abaixo está representado o fluxo de uma requisição para salvar um cartão de um cliente durante uma venda, seguido de outro fluxo onde o mesmo cliente realiza uma compra via CARTÃO PROTEGIDO.

Com a permissão do cliente para salvar seu cartão, o estabelecimento deve enviar o parâmetro `Payment.SaveCard`. Com isso será retornado um parâmetro `Credicard.CreditCardToken`.

Para mais detalhes consulte a seguinte seção do manual do Pagador:

[Salvando e Reutilizando cartões](https://braspag.github.io//manual/braspag-pagador#salvando-um-cart%C3%A3o-durante-uma-autoriza%C3%A7%C3%A3o)

Quando o cliente voltar ao site para fazer uma nova compra e se logar, o site pode apresentar a opção de “compra com 1 clique”, e o fluxo será:

1. Chamar a autorização da transação direto pela plataforma do PAGADOR, passando o `Credicard.CreditCardToken` dentro do nó `CreditCard`.
2. Receber o resultado da autorização

Você pode observar um exemplo dessa função no manual do PAGADOR também em [Criando uma transação com Card Token](https://braspag.github.io//manual/braspag-pagador#criando-uma-transa%C3%A7%C3%A3o-com-card-token)

# SALVANDO O CARTÃO DIRETAMENTE NO CARTÃO PROTEGIDO

Abaixo estão representados os fluxos de uma requisição para salvar um cartão de um cliente via CARTÃO PROTEGIDO, porém sem a necessidade de realizar uma autorização junto ao adquirente.

<aside class="request"><span class="method POST">POST</span> <span class="endpoint">/v1/Token</span></aside>

### Requisição
```json

{
	"Alias":"CartaoTesteBP",
    "Card": {
        "Number": "4929104362976003",
        "Holder": "José da Silva",
        "ExpirationDate": "12/2021",
        "SecurityCode": "123"
    }
}

```

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Payment.Alias`|Texto|64|Não |Alias (Apelido) do cartão de crédito|
|`CreditCard.Number`|Texto|16|Sim|Número do Cartão do comprador|
|`CreditCard.Holder`|Texto|25|Sim|Nome do Comprador impresso no cartão|
|`CreditCard.ExpirationDate`|Texto|7|Sim|Data de validade impresso no cartão, no formato MM/AAAA|
|`CreditCard.SecurityCode`|Texto|4|Sim|Código de segurança impresso no verso do cartão|

### Resposta

```json
{
    "Alias": "CartaoTesteBP",
    "TokenReference": "cb8ca955-ec1b-4965-98be-bd0a895a739e",
    "ExpirationDate": "2021-12-31",
    "Card": {
        "Number": "************6003",
        "ExpirationDate": "12/2021",
        "Holder": "José da Silva",
        "SecurityCode": "***"
    },
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "HRef": "https://cartaoprotegidoapisandbox.braspag.com.br/v1/Token/cb8ca955-ec1b-4965-98be-bd0a895a739e"
        }
    ]
}
```
|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`Payment.Alias`|Texto|64|Não |Alias (Apelido) do cartão de crédito|
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Card.Number`|Texto|16|Sim|Número do Cartão do comprador|
|`Card.Holder`|Texto|25|Sim|Nome do Comprador impresso no cartão|
|`Card.ExpirationDate`|Texto|7|Sim|Data de validade impresso no cartão, no formato MM/AAAA|
|`Card.SecurityCode`|Texto|4|Sim|Código de segurança impresso no verso do cartão|

# MÉTODOS DO CARTÃO PROTEGIDO

Abaixo estão representados os fluxos dos webmethods da plataforma do CARTÃO PROTEGIDO, para execução dos procedimentos via webservice.

## RECUPERANDO os dados do Cartão de Crédito

O método a seguir deve ser chamado para consultar os dados de um cartão de crédito de forma PCI Compliance, ou seja, apenas o número mascarado do cartão é retornado, juntamente com as demais informações não sensíveis.

<aside class="request"><span class="method GET">GET</span> <span class="endpoint">/v1/Token/{TokenReference}</span></aside>

### Requisição

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|


### Resposta

```json
{
    "Alias": "CartaoTesteBP",
    "TokenReference": "cb8ca955-ec1b-4965-98be-bd0a895a739e",
    "ExpirationDate": "2021-12-31",
    "Card": {
        "Number": "************6003",
        "ExpirationDate": "12/2021",
        "Holder": "José da Silva",
        "SecurityCode": "***"
    },
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "HRef": "https://cartaoprotegidoapisandbox.braspag.com.br/v1/Token/cb8ca955-ec1b-4965-98be-bd0a895a739e"
        }
    ]
}
```
|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`Payment.Status`|Texto|10|Não |Status atual do token no Cartão Protegido. Valores possíveis: Active, Removed, Suspended
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Card.Number`|Texto|16|Sim|Número do Cartão do comprador|
|`Card.Holder`|Texto|25|Sim|Nome do Comprador impresso no cartão|
|`Card.ExpirationDate`|Texto|7|Sim|Data de validade impresso no cartão, no formato MM/AAAA|
|`Card.SecurityCode`|Texto|4|Sim|Código de segurança impresso no verso do cartão|

## RECUPERANDO o TokenReference pelo Alias

O método a seguir deve ser chamado para consultar o TokenReference quando se tem apenas o Alias salvo.

<aside class="request"><span class="method GET">GET</span> <span class="endpoint">/v1/Alias/{Alias}/TokenReference</span></aside>

### Requisição

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Payment.Alias`|Texto|64|Não |Alias (Apelido) do cartão de crédito|

### Resposta

```json
{
    "TokenReference": "bcad40d4-6991-4837-b90a-34a2bbf39c51"
}
```
|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|

## INVALIDANDO um TokenReference

O método a seguir deve ser chamado para remover um token gravado. Essa exclusão é definitiva e não há forma de reverte-la.

<aside class="request"><span class="method DELETE">DELETE</span> <span class="endpoint">/v1/Token/{TokenReference}</span></aside>

### Requisição
```json
{
	"RemovedBy":"Merchant",
	"Reason":"FraudSuspicion"
}
```

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Payment.RemovedBy`|Texto|10|Sim|Discriminação de quem removeu o token. Valores possíveis: 'Merchant' ou 'CardHolder'|
|`Payment.Reason`|Texto|10|Sim|Razão da exclusão do token. Valores possíveis: 'FraudSuspicion' ou 'Other'|


### Resposta

```json
{
    "TokenReference": "53d3e737-b10d-4198-9cb3-58e07a541d4e",
    "Status": "Removed",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "HRef": "https://cartaoprotegidoapisandbox.braspag.com.br/v1/Token/53d3e737-b10d-4198-9cb3-58e07a541d4e"
        }
    ]
}
```
|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Payment.Status`|Texto|10|Não |Status atual do token no Cartão Protegido.

## SUSPENDENDO um TokenReference

O método a seguir deve ser chamado para suspender um token gravado.

<aside class="request"><span class="method PUT">PUT</span> <span class="endpoint">/v1/Token/{TokenReference}/suspend</span></aside>

### Requisição
```json
{
	"RemovedBy":"Merchant",
	"Reason":"FraudSuspicion"
}
```

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Payment.RemovedBy`|Texto|10|Sim|Discriminação de quem removeu o token. Valores possíveis: 'Merchant' ou 'CardHolder'|
|`Payment.Reason`|Texto|10|Sim|Razão da exclusão do token. Valores possíveis: 'FraudSuspicion' ou 'Other'|


### Resposta

```json
{
    "TokenReference": "53d3e737-b10d-4198-9cb3-58e07a541d4e",
    "Status": "Suspended",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "HRef": "https://cartaoprotegidoapisandbox.braspag.com.br/v1/Token/53d3e737-b10d-4198-9cb3-58e07a541d4e"
        }
    ]
}
```
|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Payment.Status`|Texto|10|Não |Status atual do token no Cartão Protegido.

## REATIVANDO um token suspenso

O método a seguir deve ser chamado para suspender um token gravado.

<aside class="request"><span class="method PUT">PUT</span> <span class="endpoint">/v1/Token/{TokenReference}/suspend</span></aside>

### Requisição
```json
{
	"RemovedBy":"Merchant",
	"Reason":"FraudSuspicion"
}
```

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Payment.RemovedBy`|Texto|10|Sim|Discriminação de quem removeu o token. Valores possíveis: 'Merchant' ou 'CardHolder'|
|`Payment.Reason`|Texto|10|Sim|Razão da exclusão do token. Valores possíveis: 'FraudSuspicion' ou 'Other'|


### Resposta

```json
{
    "TokenReference": "53d3e737-b10d-4198-9cb3-58e07a541d4e",
    "Status": "Active",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "HRef": "https://cartaoprotegidoapisandbox.braspag.com.br/v1/Token/53d3e737-b10d-4198-9cb3-58e07a541d4e"
        }
    ]
}
```
|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Payment.Status`|Texto|10|Não |Status atual do token no Cartão Protegido.

# CÓDIGO DE SEGURANÇA (CVV)

O código de segurança é obrigatório para que uma autorização, em compras não presenciais, seja aceita pelo banco emissor do cartão. Ele é mais um mecanismo de segurança no processo anti-fraude, onde busca-se validar que a pessoa que está utilizando o cartão seja de fato a dona dele. Por isso, as regras da indústria de cartões (PCI) permitem que se armazene o número do cartão e a validade, mas nunca o código de segurança. Este deve ser sempre solicitado no ato da compra para validação. Sendo a BRASPAG uma empresa PCI compliance, ela também não armazena o código de segurança, que deverá ser solicitado pelo estabelecimento no ato da confirmação da venda via CARTÃO PROTEGIDO, caso seja obrigatório o uso na Adquirente em questão.

Estabelecimentos que possuem o modelo de negócio baseado em recorrência, como, por exemplo, assinaturas de serviços, já possuem afiliação liberada para uso sem CVV.

<aside class="notice">Esta condição de recorrência é concedida exclusivamente pelas Adquirentes, não dependendo da BRASPAG.</aside>

# DICAS DE IMPLEMENTAÇÃO

## IMPLEMENTANDO ONECLICK (COMPRA COM 1 CLIQUE)

Para fazer uma venda através de um clique, é necessário que o estabelecimento já possua uma autorização, fornecida pelo cliente, para poder armazenar os dados de seu cartão de crédito. Desta forma, nas próximas compras ele pode optar por fazer o pagamento com o cartão de crédito previamente salvo. Concedida a autorização para o armazenamento, basta que o estabelecimento envie os dados do cartão para a plataforma do CARTÃO PROTEGIDO, recebendo como resposta uma chave que representa a dupla “cartão de crédito-cliente”. Para cada cartão distinto que o cliente autorize o armazenamento, o CARTÃO PROTEGIDO fornecerá uma chave também distinta.

Nas próximas vendas para este cliente, o estabelecimento poderá oferecer a “compra com 1 clique” como forma de pagamento. Isto pode ser feito através de um botão de “comprar com um clique” no produto/serviço selecionado, ou como mais um meio de pagamento no processo de finalização do carrinho de compras. Para processar uma venda via “compra com 1 clique”, basta que seja passado para a plataforma do CARTÃO PROTEGIDO a chave previamente fornecida que identifique o “cartão de crédito-cliente” e a plataforma irá autorizar direto a transação na Adquirente via PAGADOR.

### BOAS PRÁTICAS

Salvar o número do cartão mascarado para apresentar ao cliente qual cartão ele tem habilitado para “a compra com 1 clique” no site;

* Opcionalmente, também salvar a data de validade, para ativamente comunicar ao cliente que o cartão que ele tem armazenado expirou e sugerir a troca;
* Apenas salvar o cartão na plataforma do CARTÃO PROTEGIDO caso ele tenha sido autorizado com sucesso na última compra do cliente;
* Segurança do login e senha dos usuarios do site – senhas muito fracas são facilmente descobertas e o fraudador consegue fazer uma compra mesmo sem ter o cartão (no caso de não solicitação do CVV pelo site);
* Controlar variáveis de sessão para evitar que o usuário (login do cliente) permaneça logado no site e outra pessoa acesse depois fazendo “compras via 1 clique” com este login (ex: usuários conectados em lan houses).

## IMPLEMENTANDO COBRANÇA RECORRENTE

Para cada pedido a ser cobrado com recorrência de cartão de crédito, o estabelecimento deve salvar os dados do cartão de crédito na plataforma do CARTÃO PROTEGIDO e receber a chave que representa aquele “pedido-cartão”. Chegado o dia da cobrança da recorrência, basta que o método de autorização de cartão seja chamado, passando os dados para pagamento e, ao invés dos dados do cartão (número + data de validade), a chave que o representa.

Se houver necessidade de troca de cartão para determinado pedido, basta que este novo cartão seja salvo no CARTÃO PROTEGIDO, e a nova chave gerada seja associada ao pedido na plataforma do estabelecimento. Não há necessidade de cancelamento/ exclusão do cartão na plataforma.

Se houver a necessidade de associar um Alias já existente a um novo token, basta desabilitar o token antigo para deixar o Alias associado a ele, liberado para uma nova associação.

Se o estabelecimento optar por não processar a autorização da transação pela integração PAGADOR/CARTÃO PROTEGIDO, é fundamental que em nenhum momento o número do cartão seja gravado (persistido em banco ou em seção do browser) para que a segurança das informações seja mantida.

# MAPA DE ERROS

Abaixo segue a lista dos possíveis erros retornado pelos métodos no campo “ErrorReportCollection”

{AGUARDANDO O MAPA DOS DEVS}
