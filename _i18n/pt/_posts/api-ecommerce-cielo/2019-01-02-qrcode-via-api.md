---
layout: manual
title: QRCode via API E-Commerce
description: Manual de Integração QRCode via API E-Commerce
search: true
translated: true
toc_footers: true
categories: manual
sort_order: 1
tags:
  - API e-Commerce Cielo
language_tabs:
  json: JSON
  shell: cURL
---

# Visão geral - API Cielo eCommerce

O objetivo desta documentação é orientar o desenvolvedor sobre como integrar com a API E-Commerce da Cielo para gerar o QRCode de Pagamento, descrevendo as funcionalidades, os métodos a serem utilizados, listando informações a serem enviadas e recebidas, e provendo exemplos.

O mecanismo de integração com o Cielo eCommerce é simples, de modo que apenas conhecimentos intermediários em linguagem de programação para Web, requisições HTTP/HTTPS e manipulação de arquivos JSON, são necessários para implantar a solução Cielo eCommerce com sucesso.

Nesse manual você encontrará a referência sobre todas as operações disponíveis na API REST da API Cielo eCommerce. Estas operações devem ser executadas utilizando sua chave específica (Merchant ID e Merchant Key) nos respectivos endpoints dos ambientes:

Ambiente Produção

* **Requisição de transação**: https://api.cieloecommerce.cielo.com.br/
* **Consulta transação**: https://apiquery.cieloecommerce.cielo.com.br/

Ambiente Sandbox

* **Requisição de transação**: https://apisandbox.cieloecommerce.cielo.com.br
* **Consulta transação**: https://apiquerysandbox.cieloecommerce.cielo.com.br

Para executar uma operação, combine a URL base do ambiente com a URL da operação desejada e envie utilizando o verbo HTTP conforme descrito na operação.

## Arquitetura

A integração é realizada através de serviços disponibilizados como Web Services. O modelo empregado é bastante simples: Existem duas URLs (endpoint), uma específica operações que causam efeitos colaterais - como autorização, captura e cancelamento de transações, e uma URL específica para operações que não causam efeitos colaterais, como pesquisa de transações. Essas duas URLs receberão as mensagens HTTP através dos métodos POST, GET ou PUT. Cada tipo de mensagem deve ser enviada para um recurso identificado através do path.

|Método|Descrição|
|---|---|
|**POST**|O método HTTP `POST` é utilizado na criação dos recursos ou no envio de informações que serão processadas. Por exemplo, criação de uma transação.|
|**PUT**|O método HTTP `PUT` é utilizado para atualização de um recurso já existente. Por exemplo, captura ou cancelamento de uma transação previamente autorizada.|
|**GET**|O método HTTP `GET` é utilizado para consultas de recursos já existentes. Por exemplo, consulta de transações.|

## Produtos e Bandeiras suportadas

QR Code de Pagamento possui suporte às seguintes bandeiras e produtos:

| Bandeira         | Crédito à vista | Crédito parcelado Loja | Débito |
|------------------|-----------------|------------------------|--------|
| Visa             | Sim             | Sim                    | *Não*  |
| Master Card      | Sim             | Sim                    | *Não*  |
| American Express | Sim             | Sim                    | *Não*  |
| Elo              | Sim             | Sim                    | *Não*  |
| Diners Club      | Sim             | Sim                    | *Não*  |
| Discover         | Sim             | *Não*                  | *Não*  |
| JCB              | Sim             | Sim                    | *Não*  |
| Aura             | Sim             | Sim                    | *Não*  |
| Hipercard        | Sim             | Sim                    | *Não*  |

# Sandbox e Ferramentas

## Sobre o Sandbox

Para facilitar os testes durante a integração, a Cielo oferece um ambiente Sandbox que é composto por duas áreas:

1. Cadastro de conta de testes
2. Endpoints transacionais

|**Requisição**| https://apisandbox.cieloecommerce.cielo.com.br     |
| **Consulta** | https://apiquerysandbox.cieloecommerce.cielo.com.br|

**Vantagens de utilizar o Sandbox**

* Não é necessário uma afiliação para utilizar o Sandbox Cielo.
* Basta acessar o [**Cadastro do Sandbox**](https://cadastrosandbox.cieloecommerce.cielo.com.br/) criar uma conta.
* com o cadastro você receberá um `MerchantId` e um `MerchantKey`,que são as credenciais necessarias para os métodos da API

## Ferramenta para Integração: POSTMAN

O **Postman** é um API Client que facilita aos desenvolvedores criar, compartilhar, testar e documentar APIs. Isso é feito, permitindo aos usuários criar e salvar solicitações HTTPs simples e complexas, bem como ler suas respostas.

A Cielo oferece coleções completas de suas integrações via Postamn, o que facilita o processo de integração com a API Cielo.

Sugerimos que desenvolvedores acessem nosso [**Tutorial**](https://developercielo.github.io/tutorial/postman) sobre a ferramenta para compreender melhor todas as vantagens que ela oferece.

## Cartão de crédito - Sandbox

Para testar o cenário de autorização com sucesso via QRCode, utilize o cartão 4551.8700.0000.0183

As informações de **Cód.Segurança (CVV)** e validade podem ser aleatórias, mantendo o formato - CVV (3 dígitos) Validade (MM/YYYY).

## Gerando um QRCode via API

### Transação Simples

Para criar uma transação que utilizará cartão de crédito, é necessário enviar uma requisição utilizando o método `POST` para o recurso Payment, conforme o exemplo. Esse exemplo contempla o mínimo de campos necessários a serem enviados para a autorização.

<aside class="notice"><strong>Atenção:</strong> Não é possivel realizar uma transação com valor (`Amount`) 0.</aside>

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/sales/</span></aside>

```json
{  
   "MerchantOrderId":"2019010101",
   "Customer":{  
      "Name":"QRCode Test"
   },
   "Payment":{
    "isQrCode": true,   
     "Type":"CreditCard",
     "Amount":100,
     "Installments":1,
     "Capture":false
   }
}
```

```shell
curl
--request POST "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/"
--header "Content-Type: application/json"
--header "MerchantId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--header "MerchantKey: 0123456789012345678901234567890123456789"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
{  
   "MerchantOrderId":"2019010101",
   "Customer":{  
      "Name":"QRCode Test"
   },
   "Payment":{
    "isQrCode": true,   
     "Type":"CreditCard",
     "Amount":100,
     "Installments":1,
     "Capture":false
   }
}
--verbose
```

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|---|---|---|---|---|
|`MerchantId`|Guid|36|Sim|Identificador da loja na Cielo.|
|`MerchantKey`|Texto|40|Sim|Chave Publica para Autenticação Dupla na Cielo.|
|`RequestId`|Guid|36|Não|Identificador do Request, utilizado quando o lojista usa diferentes servidores para cada GET/POST/PUT.|
|`MerchantOrderId`|Texto|50|Sim|Numero de identificação do Pedido.|
|`Customer.Name`|Texto|255|Não|Nome do Comprador.|
|`Payment.isQrCode`|Booleano|-|Sim|Enviar **true** para uma transação de QRCode de Pagamento|
|`Payment.Type`|Texto|100|Sim|Tipo do Meio de Pagamento. Enviar **CreditCard** para uma transação de QRCode.|
|`Payment.Amount`|Número|15|Sim|Valor do Pedido (ser enviado em centavos).|
|`Payment.Installments`|Número|2|Sim|Número de Parcelas.|
|`Payment.Capture`|Booleano|-|Não|Enviar **true** para uma trasação de captura automática.|

#### Resposta

```json
{
    "MerchantOrderId": "2019010101",
    "Customer": {
        "Name": "QRCode Test"
    },
    "Payment": {
        "ServiceTaxAmount": 0,
        "Installments": 1,
        "Interest": 0,
        "Capture": false,
        "Authenticate": false,
        "Recurrent": false,
        "Provider": "Cielo",
        "Amount": 100,
        "ReceivedDate": "2019-01-02 10:14:29",
        "Status": 12,
        "IsSplitted": false,
        "IsQrCode": true,
        "QrCode": "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",
        "ReturnMessage": "QRCode gerado com sucesso",
        "PaymentId": "5d7e8fd3-70b6-4a88-9660-e064d72fdcdd",
        "Type": "CreditCard",
        "Currency": "BRL",
        "Country": "BRA",
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/5d7e8fd3-70b6-4a88-9660-e064d72fdcdd"
            }
        ]
    }
}
```

```shell
--header "Content-Type: application/json"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
{
    "MerchantOrderId": "2019010101",
    "Customer": {
        "Name": "QRCode Test"
    },
    "Payment": {
        "ServiceTaxAmount": 0,
        "Installments": 1,
        "Interest": 0,
        "Capture": false,
        "Authenticate": false,
        "Recurrent": false,
        "Provider": "Cielo",
        "Amount": 100,
        "ReceivedDate": "2019-01-02 10:14:29",
        "Status": 12,
        "IsSplitted": false,
        "IsQrCode": true,
        "QrCode": "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",
        "ReturnMessage": "QRCode gerado com sucesso",
        "PaymentId": "5d7e8fd3-70b6-4a88-9660-e064d72fdcdd",
        "Type": "CreditCard",
        "Currency": "BRL",
        "Country": "BRA",
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/5d7e8fd3-70b6-4a88-9660-e064d72fdcdd"
            }
        ]
    }
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|---|---|---|---|---|
|`isQrCode`|Indica se é uma transação de geração de QRCode de Pagamento|Booleano|-|**true** ou **false**|
|`QrCode`|QRCode codificado na base 64|Texto|variável|Texto alfanumérico|
|`PaymentId`|Campo Identificador do Pedido, necessário para futuras operações como Consulta, Captura e Cancelamento.|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Status`|Status da Transação. No caso de uma transação de geração de QRCode de pagamento, o status inicial é 12 (Pending).|Byte|---|2|
|`ReturnCode`|Código de retorno da Adquirência.|Texto|32|Texto alfanumérico|
|`ReturnMessage`|Mensagem de retorno da Adquirência.|Texto|512|Texto alfanumérico|

<aside class="notice">Exemplo do código HTML para apresentar uma imagem codificada:<BR><![CDATA[<img src="data:image/jpeg;base64,/9j/4RiDRXhpZgAATU0AKgA..." width="100" height="50" alt="base64 test">]]></aside>

# Consulta - Captura - Cancelamento

## Consulta de transações

### Consulta - PaymentID

Para consultar uma venda de cartão de crédito, é necessário fazer um GET para o recurso Payment conforme o exemplo.

#### Requisição

<aside class="request"><span class="method get">GET</span> <span class="endpoint">/1/sales/{PaymentId}</span></aside>

```shell
curl
--request GET "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}"
--header "Content-Type: application/json"
--header "MerchantId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--header "MerchantKey: 0123456789012345678901234567890123456789"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
--verbose
```

|Propriedade|Descrição|Tipo|Tamanho|Obrigatório|
|---|---|---|---|---|
|`MerchantId`|Identificador da loja na API Cielo eCommerce.|Guid|36|Sim|
|`MerchantKey`|Chave Publica para Autenticação Dupla na API Cielo eCommerce.|Texto|40|Sim|
|`RequestId`|Identificador do Request, utilizado quando o lojista usa diferentes servidores para cada GET/POST/PUT|Guid|36|Não|
|`PaymentId`|Numero de identificação do Pagamento.|Texto|36|Sim|

#### Resposta

```json
{
    "MerchantOrderId": "2014111706",
    "Customer": {
        "Name": "Comprador Teste",
        "Address": {}
    },
    "Payment": {
        "ServiceTaxAmount": 0,
        "Installments": 1,
        "Interest": "ByMerchant",
        "Capture": false,
        "Authenticate": false,
        "CreditCard": {
            "CardNumber": "455187******0183",
            "Holder": "Teste Holder",
            "ExpirationDate": "12/2030",
            "SaveCard": false,
            "Brand": "Visa"
        },
        "ProofOfSale": "674532",
        "AuthorizationCode": "123456",
        "PaymentId": "24bc8366-fc31-4d6c-8555-17049a836a07",
        "Type": "CreditCard",
        "Amount": 15700,
        "Currency": "BRL",
        "Country": "BRA",
        "ExtraDataCollection": [],
        "Status": 1,
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}"
            },
            {
                "Method": "PUT",
                "Rel": "capture",
                "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}/capture"
            },
            {
                "Method": "PUT",
                "Rel": "void",
                "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}/void"
            }
        ]
    }
}
```

```shell
--header "Content-Type: application/json"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
{
    "MerchantOrderId": "2014111706",
    "Customer": {
        "Name": "Comprador Teste",
        "Address": {}
    },
    "Payment": {
        "ServiceTaxAmount": 0,
        "Installments": 1,
        "Interest": "ByMerchant",
        "Capture": false,
        "Authenticate": false,
        "CreditCard": {
            "CardNumber": "455187******0183",
            "Holder": "Teste Holder",
            "ExpirationDate": "12/2030",
            "SaveCard": false,
            "Brand": "Visa"
        },
        "ProofOfSale": "674532",
        "AuthorizationCode": "123456",
        "PaymentId": "24bc8366-fc31-4d6c-8555-17049a836a07",
        "Type": "CreditCard",
        "Amount": 15700,
        "Currency": "BRL",
        "Country": "BRA",
        "ExtraDataCollection": [],
        "Status": 1,
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}"
            },
            {
                "Method": "PUT",
                "Rel": "capture",
                "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}/capture"
            },
            {
                "Method": "PUT",
                "Rel": "void",
                "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}/void"
            }
        ]
    }
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|---|---|---|---|---|
|`ProofOfSale`|Número da autorização, identico ao NSU.|Texto|6|Texto alfanumérico|
|`AuthorizationCode`|Código de autorização.|Texto|6|Texto alfanumérico|
|`PaymentId`|Campo Identificador do Pedido.|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`Status`|Status da Transação.|Byte|---|2|
|`Customer.Name`|Texto|255|Não|Nome do Comprador.|
|`Customer.Status`|Texto|255|Não|Status de cadastro do comprador na loja (NEW / EXISTING)|
|`Payment.Type`|Texto|100|Sim|Tipo do Meio de Pagamento.|
|`Payment.Amount`|Número|15|Sim|Valor do Pedido (ser enviado em centavos).|
|`Payment.Provider`|Texto|15|---|Define comportamento do meio de pagamento (ver Anexo)/NÃO OBRIGATÓRIO PARA CRÉDITO.|
|`Payment.Installments`|Número|2|Sim|Número de Parcelas.|
|`CreditCard.CardNumber`|Texto|19|Sim|Número do Cartão do Comprador.|
|`CreditCard.Holder`|Texto|25|Não|Nome do Comprador impresso no cartão.|
|`CreditCard.ExpirationDate`|Texto|7|Sim|Data de validade impresso no cartão.|
|`CreditCard.SecurityCode`|Texto|4|Não|Código de segurança impresso no verso do cartão - Ver Anexo.|
|`CreditCard.Brand`|Texto|10|Não|Bandeira do cartão (Visa / Master / Amex / Elo / Aura / JCB / Diners / Discover / Hipercard).|

### Consulta - MerchandOrderID

Não é possível consultar diretamente uma pagamento pelo identificador enviado pela loja (MerchantOrderId), mas é possível obter todos os PaymentIds associados ao identificador.

Para consultar uma venda pelo identificador da loja, é necessário fazer um GET para o recuso sales conforme o exemplo.

#### Requisição

<aside class="request"><span class="method get">GET</span> <span class="endpoint">/1/sales?merchantOrderId={merchantOrderId}</span></aside>

```shell
curls
--request GET " https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales?merchantOrderId={merchantOrderId}"
--header "Content-Type: application/json"
--header "MerchantId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--header "MerchantKey: 0123456789012345678901234567890123456789"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
--verbose
```

|Propriedade|Descrição|Tipo|Tamanho|Obrigatório|
|---|---|---|---|---|
|`MerchantId`|Identificador da loja na API Cielo eCommerce.|Guid|36|Sim|
|`MerchantKey`|Chave Publica para Autenticação Dupla na API Cielo eCommerce.|Texto|40|Sim|
|`RequestId`|Identificador do Request, utilizado quando o lojista usa diferentes servidores para cada GET/POST/PUT|Guid|36|Não|
|`MerchantOrderId`|Campo Identificador do Pedido na Loja.|Texto|36|Sim|

#### Resposta

```json
{
    "Payment": [
        {
            "PaymentId": "5fb4d606-bb63-4423-a683-c966e15399e8",
            "ReceveidDate": "2015-04-06T10:13:39.42"
        },
        {
            "PaymentId": "6c1d45c3-a95f-49c1-a626-1e9373feecc2",
            "ReceveidDate": "2014-12-19T20:23:28.847"
        }
    ]
}
```

```shell
--header "Content-Type: application/json"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
{
    "Payment": [
        {
            "PaymentId": "5fb4d606-bb63-4423-a683-c966e15399e8",
            "ReceveidDate": "2015-04-06T10:13:39.42"
        },
        {
            "PaymentId": "6c1d45c3-a95f-49c1-a626-1e9373feecc2",
            "ReceveidDate": "2014-12-19T20:23:28.847"
        }
    ]
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|---|---|---|---|---|
|`PaymentId`|Campo Identificador do Pedido.|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|

## Captura

A **Captura** é passo exclusivo para transações de Cartões de Crédito.

Ao realizar uma captura, o lojita confirma que o valor autorizado no cartão poderá ser cobrado pela insituição financeira emissora do cartão.

O que a captura gera:

* Ela executa a cobrança do cartão
* Ela inclui o valor da venda na fatura do comprador
* Somente transações capturadas são pagas pela Cielo ao lojista

<aside class="notice"><strong>Atenção:</strong> A captura é um processo com prazo de execução. Verifique em sem cadastro cielo qual o limite habilitado para a sua afiliação. Após esse periodo, não é possivel realiza a Captura da transação</aside>

<aside class="notice"><strong>Atenção:</strong> Captura parcial disponível apenas para transações de crédito</aside>

### Requisição - Captura Parcial

<aside class="request"><span class="method put">PUT</span> <span class="endpoint">/1/sales/{paymentId}/capture?amount={Valor}</span></aside>

```json

https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{paymentId}/capture?amount={Valor}

```

```shell
curl
--request PUT "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{paymentId}/capture?amount={Valor}"
--header "Content-Type: application/json"
--header "MerchantId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--header "MerchantKey: 0123456789012345678901234567890123456789"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--verbose
```

|Propriedade|Descrição|Tipo|Tamanho|Obrigatório|
|---|---|---|---|---|
|`MerchantId`|Identificador da loja na API Cielo eCommerce.|Guid|36|Sim|
|`MerchantKey`|Chave Publica para Autenticação Dupla na API Cielo eCommerce.|Texto|40|Sim|
|`RequestId`|Identificador do Request, utilizado quando o lojista usa diferentes servidores para cada GET/POST/PUT|Guid|36|Não|
|`PaymentId`|Campo Identificador do Pedido.|Guid|36|Sim|
|`Amount`|Valor do Pedido (ser enviado em centavos).|Número|15|Não|
|`ServiceTaxAmount`|[Veja Anexo](https://developercielo.github.io/manual/cielo-ecommerce#service-tax-amount-taxa-de-embarque)|Número|15|Não|

### Resposta

```json
{
    "Status": 2,
    "ReasonCode": 0,
    "ReasonMessage": "Successful",
    "ProviderReturnCode": "6",
    "ProviderReturnMessage": "Operation Successful",
    "ReturnCode": "6",
    "ReturnMessage": "Operation Successful",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/8b1d43ee-a918-40d2-ba62-e5665e7ccbd3"
        },
        {
            "Method": "PUT",
            "Rel": "void",
            "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/8b1d43ee-a918-40d2-ba62-e5665e7ccbd3/void"
        }
    ]
}
```

```shell
--header "Content-Type: application/json"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
{
    "Status": 2,
    "ReasonCode": 0,
    "ReasonMessage": "Successful",
    "ProviderReturnCode": "6",
    "ProviderReturnMessage": "Operation Successful",
    "ReturnCode": "6",
    "ReturnMessage": "Operation Successful",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/8b1d43ee-a918-40d2-ba62-e5665e7ccbd3"
        },
        {
            "Method": "PUT",
            "Rel": "void",
            "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/8b1d43ee-a918-40d2-ba62-e5665e7ccbd3/void"
        }
    ]
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|---|---|---|---|---|
|`Status`|Status da Transação.|Byte|---|2|
|`ReturnCode`|Código de retorno da adquirente.|Texto|32|Texto alfanumérico|
|`ReturnMessage`|Mensagem de retorno da adquirente.|Texto|512|Texto alfanumérico|
|`ProviderReturnCode`|Código de retorno do Provider.|Texto|32|Texto alfanumérico|
|`ProviderReturnMessage`|Mensagem de retorno do Provider.|Texto|512|Texto alfanumérico|

<aside class="notice"><strong>Captura de Taxa de embarque</strong> Para realizar a captura da *taxa de embarque*, basta adicionar o valor do ServiveTaxAmount a ser capturado</aside>

<aside class="request"><span class="method put">PUT</span> <span class="endpoint">/1/sales/{paymentId}/capture?amount={Valor}&serviceTaxAmount=xxx</span></aside>

#### Resposta

```json
{
    "Status": 2,
    "ReasonCode": 0,
    "ReasonMessage": "Successful",
    "ProviderReturnCode": "0",
    "ProviderReturnMessage": "Operation Successful",
    "ReturnCode": "0",
    "ReturnMessage": "Operation Successful",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8"
        },
        {
            "Method": "PUT",
            "Rel": "void",
            "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8/void"
        }
    ]
}
```

```shell
--header "Content-Type: application/json"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
{
    "Status": 2,
    "ReasonCode": 0,
    "ReasonMessage": "Successful",
    "ProviderReturnCode": "0",
    "ProviderReturnMessage": "Operation Successful",
    "ReturnCode": "0",
    "ReturnMessage": "Operation Successful",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8"
        },
        {
            "Method": "PUT",
            "Rel": "void",
            "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8/void"
        }
    ]
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|---|---|---|---|---|
|`Status`|Status da Transação.|Byte|---|2|
|`ReturnCode`|Código de retorno da adquirente.|Texto|32|Texto alfanumérico|
|`ReturnMessage`|Mensagem de retorno da adquirente.|Texto|512|Texto alfanumérico|
|`ProviderReturnCode`|Código de retorno do Provider.|Texto|32|Texto alfanumérico|
|`ProviderReturnMessage`|Mensagem de retorno do Provider.|Texto|512|Texto alfanumérico|

## Cancelamento

O **cancelamento** é a operação responsável pela cancelamento total ou parcial de um valor autorizado ou capturado.

Basta realizar um `POST` enviando o valor a ser cancelado.

<aside class="notice"><strong>Atenção:</strong> Cancelamento parcial é disponível apenas para transações de crédito *CAPTURADAS*</aside>

<aside class="notice"><strong>Atenção:</strong> O retorno da API soma o total de cancelamentos Parciais, ou seja, se 3 cancelamentos de R$10,00 forem realizados, a API apresentará em seu retorno um total de R$30,00 cancelados</aside>

### Requisição - cancelamento

<aside class="request"><span class="method put">PUT</span> <span class="endpoint">/1/sales/{PaymentId}/void?amount=XXX </span></aside>

```shell
curl
--request PUT "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}/void?amount=XXX"
--header "Content-Type: application/json"
--header "MerchantId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--header "MerchantKey: 0123456789012345678901234567890123456789"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--verbose
```

|Propriedade|Descrição|Tipo|Tamanho|Obrigatório|
|---|---|---|---|---|
|`MerchantId`|Identificador da loja na API Cielo eCommerce.|Guid|36|Sim|
|`MerchantKey`|Chave Publica para Autenticação Dupla na API Cielo eCommerce.|Texto|40|Sim|
|`RequestId`|Identificador do Request, utilizado quando o lojista usa diferentes servidores para cada GET/POST/PUT|Guid|36|Não|
|`PaymentId`|Campo Identificador do Pedido.|Guid|36|Sim|
|`Amount`|Valor do Pedido (ser enviado em centavos).|Número|15|Não|

### Resposta

```json
{
    "Status": 2,
    "ReasonCode": 0,
    "ReasonMessage": "Successful",
    "ProviderReturnCode": "0",
    "ProviderReturnMessage": "Operation Successful",
    "ReturnCode": "0",
    "ReturnMessage": "Operation Successful",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8"
        },
        {
            "Method": "PUT",
            "Rel": "void",
            "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8/void"
        }
    ]
}
```

```shell
--header "Content-Type: application/json"
--header "RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--data-binary
{
    "Status": 2,
    "ReasonCode": 0,
    "ReasonMessage": "Successful",
    "ProviderReturnCode": "0",
    "ProviderReturnMessage": "Operation Successful",
    "ReturnCode": "0",
    "ReturnMessage": "Operation Successful",
    "Links": [
        {
            "Method": "GET",
            "Rel": "self",
            "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8"
        },
        {
            "Method": "PUT",
            "Rel": "void",
            "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/4d7be764-0e81-4446-b31e-7eb56bf2c9a8/void"
        }
    ]
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|---|---|---|---|---|
|`Status`|Status da Transação.|Byte|---|2|
|`ReturnCode`|Código de retorno da adquirente.|Texto|32|Texto alfanumérico|
|`ReturnMessage`|Mensagem de retorno da adquirente.|Texto|512|Texto alfanumérico|
|`ProviderReturnCode`|Código de retorno do Provider.|Texto|32|Texto alfanumérico|
|`ProviderReturnMessage`|Mensagem de retorno do Provider.|Texto|512|Texto alfanumérico|

<aside class="notice"><strong>Cancelamento de Taxa de embarque</strong> Para realizar o cancelamento da *taxa de embarque*, basta adicionar o valor do ServiveTaxAmount a ser cancelado</aside>

```
https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{paymentId}/void?amount={Valor}&serviceTaxAmount=xxx
```
