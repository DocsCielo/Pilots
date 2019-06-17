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

# O que é Cartão Protegido?

O **Cartão Protegido** é uma plataforma que permite o armazenamento seguro de cartões de crédito e débito. Contamos com ambiente totalmente certificado pelo respeitado conselho de padrão de segurança PCI Security Standards Council, que assegura que a Braspag segue plenamente os rígidos requisitos e normas determinadas pelo mesmo.

A plataforma é compatível com o gateway Pagador, também da Braspag, facilitando o processamento de transações de cartão de crédito e débito via token.

## Casos de Uso

A plataforma tem como propósito ajudar os estabelecimentos que possuem diversos casos de usos, entre eles:

* **Cobrança Recorrente Agendada (Scheduled Recurring Payments)**: Estabelecimentos que já possuam uma solução interna de gerenciamento de recorrências podem utilizar a plataforma para armazenar os dados de cartão de crédito e processar através de tokens de pagamento. Exmeplo: assinatura de serviços. 

* **Cobrança Recorrente não Agendada (Unscheduled Recurring Payments)**: Estabelecimentos que cobram seus clientes já cadastrados, mas sem uma periodicidade definida. Exemplo: aplicativos de transporte. 

* **Compra com um clique (Just Click Payments)**: A “compra com um clique” permite que um pagamento online, via cartão de crédito, seja feito pulando a etapa de preenchimento dos dados para pagamento ou até mesmo de todo o processo do carrinho de compras, pois os dados do cartão já foram previamente informados pelo comprador em compras passadas e serão replicados em futuras compras mediante sua autorizaçãol.

* **Recuperação de vendas**: Estabelecimentos podem entrar novamente em contato com os clientes que eventualmente tiveram problemas na compra, oferecer uma nova tentativa de cobrança. 

## Arquitetura

A integração é realizada através de serviços disponibilizados como Web Services. O modelo empregado é bastante simples: Através do endpoint https://cartaoprotegidoapisandbox.braspag.com.br/v1/ serão enviadas todas as requisições relativas à esse serviço. Essa URL recebera as mensagens HTTP através dos métodos POST, GET ou Del. Cada tipo de mensagem deve ser enviada para um endereço identificado através do "path".

* **POST** - O método HTTP POST é utilizado na criação do token.
* **DEL** - O método HTTP DEL é utilizado para remoção de token.
* **GET** - O método HTTP GET é utilizado para consultas de recursos já existentes. Por exemplo, consulta de tokens já criados.

# Como se integra?

## Etapa de Autenticação

Para consumir os métodos da API, é necessário obter o AccessToken no padrão OAuth 2.0

|Ambiente | Endpoint | Authorization |
|---|---|---|
| **SANDBOX** | https://authsandbox.braspag.com.br/oauth2/token | **Basic _(Authorization)_**<br><br>O valor do Authorization deve ser obtido concatenando-se o valor do "ClientID", sinal de dois-pontos (":") e "ClientSecret"<br><br>Ex. e1c54542-4cbc-4958-b449-b73107e1f6c0:kbm/UFtg0fbh8OJEZDbOPmKSvRLIq+tjy8vS6s6ziXE=<br><br>e na sequência, codificar o resultado na base 64. <br>Com isso, será gerado um código alphanumérico que será utilizado na requisição de access token. Para efeitos de teste, utilize os dados abaixo:<br><br>ClientID: **e1c54542-4cbc-4958-b449-b73107e1f6c0**<br>ClientSecret: **kbm/UFtg0fbh8OJEZDbOPmKSvRLIq+tjy8vS6s6ziXE=**|
| --- | --- |
| **PRODUÇÃO** | https://auth.braspag.com.br/oauth2/token | Solicite os dados "ClientID" e "ClientSecret" à equipe de suporte após concluir o desenvolvimento em sandbox. |

### Request

<aside class="request"><span class="method post">POST</span> <span class="endpoint">oauth2/token</span></aside>

**Parâmetros no cabeçalho (Header)**

|Key|Value|
|---|---|
|`Content-Type`|application/x-www-form-urlencoded|
|`Authorization`|Basic _(Authorization)_|

**Parâmetros no corpo (Body)**

|Key|Value|
|---|---|
|`grant_type`|client_credentials|

### Response

**Parâmetros no corpo (Body)**

|Parâmetro|Descrição|
|---|---|
|`access_token`|O token de acesso solicitado. O aplicativo pode usar esse token para se autenticar no recurso protegido|
|`token_type`|Indica o valor do tipo de token|
|`expires_in`|Expiração do o token de acesso, em segundos <br/> O token quando expirar, é necessário obter um novo|

``` json
{
  "access_token": "faSYkjfiod8ddJxFTU3vti_ ... _xD0i0jqcw",
  "token_type": "bearer",
  "expires_in": 599
}
```

## Create Token

Abaixo estão representados os fluxos de uma requisição para salvar um cartão de um cliente via CARTÃO PROTEGIDO, porém sem a necessidade de realizar uma autorização junto ao adquirente.

### Request

<aside class="request"><span class="method post">POST</span><span class="endpoint">/v1/Token</span></aside>

```json
{
    "Alias":"CartaoDoJose",    
    "Card": {
        "Number": "4551870000000183",
        "Holder": "José da Silva",
        "ExpirationDate": "12/2021",
        "SecurityCode": "123"
    }
}
```

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Alias`|Texto|64|Não |Alias do cartão. O valor desta informação deve ser único (não pode repetir).|
|`Card.Number`|Número|16|Sim|Número do Cartão do comprador|
|`Card.Holder`|Texto|25|Sim|Nome do Comprador impresso no cartão|
|`Card.ExpirationDate`|Texto|7|Sim|Data de validade impresso no cartão, no formato MM/AAAA|
|`Card.SecurityCode`|Número|4|Sim|Código de segurança impresso no verso do cartão|

### Response

```json
{
    "Alias": "CartaoDoJose",
    "TokenReference": "cb8ca955-ec1b-4965-98be-bd0a895a739e",
    "ExpirationDate": "2021-12-31",
    "Card": {
        "Number": "************0183",
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
|`Alias`|Texto|64|Não |Alias (Apelido) do cartão de crédito|
|`TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`ExpirationDate`|Data de expiração do token, no formato MM/AAAA|Texto|7|MM/AAAA|
|`Card.Number`|Número|16|Sim|Número do Cartão do comprador|
|`Card.Holder`|Texto|25|Sim|Nome do Comprador impresso no cartão|
|`Card.ExpirationDate`|Texto|7|Sim|Data de validade impresso no cartão, no formato MM/AAAA|
|`Card.SecurityCode`|Número|4|Sim|Código de segurança impresso no verso do cartão|

## Get Token

### Request

<aside class="request"><span class="method get">GET</span> <span class="endpoint">/v1/Token/{TokenReference}</span></aside>

**Parâmetros no cabeçalho (Header)**

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`Payment.TokenReference`|GUID|36|Sim|Token no Cartão Protegido que representa os dados do cartão|

### Response

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
|`Payment.Status`|Status atual do token no Cartão Protegido.|-|Valores possíveis: Active, Removed, Suspended|Texto|
|`Payment.TokenReference`|Token no Cartão Protegido que representa os dados do cartão|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Card.Number`|Número do Cartão do comprador|Número|16|Exemplo: 4551870000000183|
|`Card.Holder`|Nome do Comprador impresso no cartão, sem caraceteres acentuados|Texto|25|Exemplo: Jose da Silva|
|`Card.ExpirationDate`|Data de validade impresso no cartão, no formato MM/AAAA|Texto|7|Exemplo: 12/2021|
|`Card.SecurityCode`|Código de segurança impresso no verso do cartão|Número|4|Exemplo: 999|

## ========================

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
