---
layout: manual
title: Integration Manual with Dynamic MCC
description: The purpose of this documentation is to guide the developer on how to integrate with Cielo's API Cielo eCommerce using Dynamic MCC resource.
search: true
translated: true
categories: manual
tags:
  - API e-Commerce Cielo
language_tabs:
  json: JSON
  shell: cURL
---

# Overview - API Cielo eCommerce

The purpose of this documentation is to guide the developer on how to integrate with Cielo's API Cielo eCommerce using the feature Dynamic MCC (Merchant Category Code), listing information to be sent and received, and providing examples.

The integration mechanism with Cielo e-Commerce is simple, so only intermediate knowledge in Web programming language, HTTP/HTTPS requests and JSON file manipulation are required to successfully deploy the Cielo e-Commerce solution.

In this manual, you will find the reference on all operations available on API REST of API Cielo eCommerce. These operations must be performed using its specific key (Merchant ID and Merchant Key) in the respective environment endpoints:

Production Environment

* **Transaction request**: https://api.cieloecommerce.cielo.com.br/
* **Transaction query**: https://apiquery.cieloecommerce.cielo.com.br/

Sandbox Environment

* **Transaction request**: https://apisandbox.cieloecommerce.cielo.com.br
* **Transaction query**: https://apiquerysandbox.cieloecommerce.cielo.com.br

To perform an operation, combine the base URL of the environment with the URL of the desired operation and send it using the HTTP verb as described in the operation.

# Requests and Responses

## Creating a transaction using Dynamic MCC

To create a transaction with Dynamic MCC, it is necessary to send a request using the `POST` method to the Payment feature as shown.

### Request

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/sales/</span></aside>

```json
{
   "merchantorderid":"7864411894",
   "customer":{
      "name":"joao da silva"
   },
   "payment":{
      "type":"creditcard",
      "amount":100,
      "currency":"brl",
      "installments":1,
      "capture":false,
      "softdescriptor":"softdescriptor",
      "Mcc":"0000",
      "creditcard":{
         "cardnumber":"4551870000000183",
         "holder":"joao da silva",
         "expirationdate":"12/2026",
         "savecard":false,
         "brand":"visa"
      }
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
   "merchantorderid":"7864411894",
   "customer":{
      "name":"joao da silva"
   },
   "payment":{
      "type":"creditcard",
      "amount":100,
      "currency":"brl",
      "installments":1,
      "capture":false,
      "softdescriptor":"softdescriptor",
      "Mcc":"0000",
      "creditcard":{
         "cardnumber":"4551870000000183",
         "holder":"joao da silva",
         "expirationdate":"12/2026",
         "savecard":false,
         "brand":"visa"
      }
   }
}
--verbose
```

|Property|Type|Size|Required|Description|
|---|---|---|---|---|
|`MerchantId`|Guid|36|Yes|Store identifier in Cielo.|
|`MerchantKey`|Text|40|Yes|Public Key for Double Authentication in Cielo.|
|`RequestId`|Guid|36|No|Request Identifier, used when the merchant uses different servers for each GET/POST/PUT.|
|`MerchantOrderId`|Text|50|Yes|Order ID number.|
|`Customer.Name`|Text|255|No|Buyer's name.|
|`Payment.Type`|Text|100|Yes|Means of Payment Type.|
|`Payment.Amount`|Number|15|Yes|Order Amount (to be sent in cents).|
|`Payment.Currency`|Text|3|No|Currency in which payment will be made (BRL).|
|`Payment.Installments`|Number|2|Yes|Number of Installments.|
|`Payment.Capture`|Boolean|---|No (Default false)|Boolean that identifies that the authorization should be with automatic capture.|
|`Payment.SoftDescriptor`|Text|13|No|Text printed on the buyer's bank invoice - Only available for VISA/MASTER - does not allow special characters|
|`Payment.Mcc`|Text|4|No|Merchant Category Code that identifies the sub-merchant's category code.|
|`CreditCard.CardNumber`|Text|19|Yes|Buyer's Card Number.|
|`CreditCard.Holder`|Text|25|No|Buyer's name printed on card.|
|`CreditCard.ExpirationDate`|Text|7|Yes|Expiry date printed on card.|
|`CreditCard.SecurityCode`|Text|4|No|Security code printed on back of card - See Annex.|
|`CreditCard.SaveCard`|Boolean|---|No (Default false)|Boolean that identifies whether the card will be saved to generate the CardToken.|
|`CreditCard.Brand`|Text|10|Yes|Card issuer (Visa / Master / Amex / Elo / Aura / JCB / Diners / Discover / Hipercard).|

### Response

```json
{
    "MerchantOrderId": "7864411894",
    "Customer": {
        "Name": "joao da silva"
    },
    "Payment": {
        "ServiceTaxAmount": 0,
        "Installments": 1,
        "Interest": "ByMerchant",
        "Capture": true,
        "Authenticate": false,
        "CreditCard": {
            "CardNumber": "455187******0183",
            "Holder": "joao da silva",
            "ExpirationDate": "12/2026",
            "SaveCard": false,
            "Brand": "Visa"
        },
        "ProofOfSale": "674532",
        "Tid": "0305020554239",
        "AuthorizationCode": "123456",
        "SoftDescriptor":"softdescriptor",
        "Mcc":"0000",
        "PaymentId": "24bc8366-fc31-4d6c-8555-17049a836a07",
        "Type": "CreditCard",
        "Amount": 100,
        "CapturedAmount": 0,
        "Country": "BRA",
        "ExtraDataCollection": [],
        "Status": 1,
        "ReturnCode": "6",
        "ReturnMessage": "Operation Successful",
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}"
            }
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
    "MerchantOrderId": "7864411894",
    "Customer": {
        "Name": "joao da silva"
    },
    "Payment": {
        "ServiceTaxAmount": 0,
        "Installments": 1,
        "Interest": "ByMerchant",
        "Capture": true,
        "Authenticate": false,
        "CreditCard": {
            "CardNumber": "455187******0183",
            "Holder": "joao da silva",
            "ExpirationDate": "12/2026",
            "SaveCard": false,
            "Brand": "Visa"
        },
        "ProofOfSale": "674532",
        "Tid": "0305020554239",
        "AuthorizationCode": "123456",
        "SoftDescriptor":"softdescriptor",
        "Mcc":"0000",
        "PaymentId": "24bc8366-fc31-4d6c-8555-17049a836a07",
        "Type": "CreditCard",
        "Amount": 100,
        "CapturedAmount": 0,
        "Country": "BRA",
        "ExtraDataCollection": [],
        "Status": 1,
        "ReturnCode": "6",
        "ReturnMessage": "Operation Successful",
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}"
            }
            {
                "Method": "PUT",
                "Rel": "void",
                "Href": "https://apisandbox.cieloecommerce.cielo.com.br/1/sales/{PaymentId}/void"
            }
        ]
    }
}
```

|Property|Description|Type|Size|Format|
|---|---|---|---|---|
|`ProofOfSale`|Authorization number, identical to NSU.|Text|6|Alphanumeric text|
|`Tid`|Transaction Id on the acquirer.|Text|20|Alphanumeric text|
|`AuthorizationCode`|Authorization code.|Text|6|Alphanumeric text|
|`SoftDescriptor`|Text that will be printed on the carrier's bank invoice - Available only for VISA/MASTER - does not allow special characters|Text|13|Alphanumeric text|
|`PaymentId`|Order Identifier Field.|Guid|36|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`ECI`|Eletronic Commerce Indicator. Represents how secure a transaction is.|Text|2|Examples: 7|
|`Status`|Transaction Status.|Byte|---|2|
|`ReturnCode`|Return code of Acquiring.|Text|32|Alphanumeric text|
|`ReturnMessage`|Return message of Acquiring.|Text|512|Alphanumeric texto|

## Test cards for sandbox enviroment

To test purpose, we are providing **test cards** in the table below.

<aside class="notice">The <code>status</code> of the transactions are defined by the FINALS of each card, as well as the <code>ReturnCode</code>.</aside>

|Transaction Status|Card End|Return Code|Return message|
|---|---|---|---|
|Authorized|0000.0000.0000.0001<br>0000.0000.0000.0004|4/6|Operation performed successfully|
|Not Authorized|0000.0000.0000.0002|05|Not Authorized|
|Not Authorized|0000.0000.0000.0003|57|Expired Card|
|Not Authorized|0000.0000.0000.0005|78|Locked Card|
|Not Authorized|0000.0000.0000.0006|99|Time Out|
|Not Authorized|0000.0000.0000.0007|77|Canceled Card||
|Not Authorized|0000.0000.0000.0008|70|Problems with the Credit Card|
|Random Authorization|0000.0000.0000.0009|99|Operation Successful / Time Out|

Example of a Test card - 4024.0071.5376.3191

The information of **Security Code (CVV)** and validity may be random, keeping the format - CVV (3 digits) Validity (MM/YYYY).

<aside class="notice"><strong>Warning:</strong> The **sandbox** environment evaluates the format and end of the card, if an actual card is sent, the result of the operation will be identical to that described in the test cards table.</aside>

<aside class="notice"><strong>Tokenization:</strong> Transactions in sandbox environment involving tokenization did not work based on test cards. Every card saved at the tokenization is treated as a real card, so it is not used in the simulation process</aside>

# API codes

API Cielo e-commerce has 4 types of returned codes that represent different moments of the transaction.

See more at https://developercielo.github.io/en/manual/cielo-ecommerce#api-codes
