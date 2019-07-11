---
layout: manual
title: Manual de Integração Solução Omni
description: API para integração de vendas no físico e OnLine
search: true
translated: true
toc_footers: true
categories: manual
sort_order: 1
tags:
  - API Pagamento
language_tabs:
  json: JSON
  shell: cURL
---

# Visão geral - API Solução Omni

| SandBox                                             | Produção                                      |
|:---------------------------------------------------:|:---------------------------------------------:|
| https://apisandbox.cieloecommerce.cielo.com.br      | https://api.cieloecommerce.cielo.com.br/      |

# Objetivo

Possibilitar a integração de parceiros de negócio/Subadquirentes com a Cielo para transações com cartões não-presentes (transações digitadas) e cartões presentes nas modalidades Chip e Tarja.

# Glossário

|ID|Descrição|
|BC|Biblioteca Compartilhada para PINPad|
|DUKPT|(Devired Unique Key Per Transaction) Método de criptografia utilizado na Cielo|
|PIN|Senha do cartão|
|BDK|(Base Derived Key) Chave do Sub a ser instalada no HSM|
|HSM|(Hardware Security Module) Servidor para geração, armazenamento, gerenciamento e funcionalidades criptográficas de chaves digitais|
|OAUTH2|Protocolo de autenticação utilizado nas APIs|

# Pré-requisitos

Para a integração é necessário que a solução de captura do parceiro de negócio/Subadquirente possua os seguintes componentes:

* Biblioteca Compartilhada para PINPad ou biblioteca proprietária certificada com as bandeiras.
* Chaves de Criptografia DUKPT implementada para PIN.
* Disponibilizar sua BDK para instalação no HSM Cielo.

Formato da Chave exigida pela Cielo:

O HSM Cielo está parametrizado para um KSN da seguinte forma:

* **KSI -** Número de identificação da Chave
* **DID –** Device ID
* **TC –** Transaction Counter

No cadastro da chave somente é inserido o KSI que possui 5 caracteres numéricos e a chave, conforme exemplo abaixo:

**FFFFF**030331234500012

<aside class="warning">Obs.: Os F’s devem ser preenchidos automaticamente pela Solução de Captura.</aside>

# Autenticação

A autenticação é uma operação necessária para obtenção do token que será utilizado nas demais chamadas de APIs.

|Security scheme type:|OAuth2|
|clientCredentials OAuth Flow|**Token URL:** https://authsandbox.braspag.com.br/oauth2/token<br><br>**Scopes:**<br><br>* `Administrator` - Admin everything<br><br>* `AnalyticsApiOverview` - See the analytics<br><br>* `AdminBackoffice` - Use the backoffice|

# Pagamento

## Cria um novo pagamento

Quando um pagamento é criado (201 - Created), deve-se analisar o Status (Payment.Status) na resposta para certificar-se que o pagamento foi gerado com sucesso ou se houve alguma falha.

### Venda com cartão de crédito digitado e sem senha

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/</span></aside>

```json
{
  "MerchantOrderId": "201904150001",
  "Payment": {
    "Type": "PhysicalCreditCard",
    "SoftDescriptor": "Description",
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "Amount": 15798,
    "Installments": 1,
    "Interest": "ByMerchant",
    "ProductId": 1,
    "CreditCard": {
      "CardNumber": 1234567812345678,
      "ExpirationDate": "12/2020",
      "SecurityCodeStatus": "Collected",
      "SecurityCode": 1230,
      "BrandId": 1,
      "IssuerId": 2,
      "InputMode": "Typed",
      "AuthenticationMethod": "NoPassword",
      "TruncateCardNumberWhenPrinting": true
    },
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    }
  }
}
```

#### Resposta

```json
{
  "MerchantOrderId": "20180204",
  "Customer": {
    "Name": "[Guest]"
  },
  "Payment": {
    "Installments": 1,
    "Interest": "ByMerchant",
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "TruncateCardNumberWhenPrinting": true,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlineAuthentication",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "ServiceTaxAmount": 0,
    "SoftDescriptor": "Description",
    "ProductId": 1,
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    },
    "Amount": 15798,
    "ReceivedDate": "2019-04-15T12:00:00Z",
    "CapturedAmount": 15798,
    "Provider": "Cielo",
    "ConfirmationStatus": 0,
    "InitializationVersion": 1558708320029,
    "EmvResponseData": "123456789ABCD1345DEA",
    "Status": 2,
    "IsSplitted": false,
    "ReturnCode": 0,
    "ReturnMessage": "Successful",
    "PaymentId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
    "Type": "PhysicalDebitCard",
    "Currency": "BRL",
    "Country": "BRA",
    "Links": [
      {
        "Method": "GET",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "DELETE",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "PUT",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/confirmation"
      }
    ],
    "PrintMessage": [
      {
        "Position": "Top",
        "Message": "Transação autorizada"
      },
      {
        "Position": "Bottom",
        "Message": "Obrigado e volte sempre!"
      }
    ]
  }
}
```

### Venda com cartão de crédito com leitura de tarja e senha

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/</span></aside>

```json
{
  "MerchantOrderId": "201904150002",
  "Payment": {
    "Type": "PhysicalCreditCard",
    "SoftDescriptor": "Description",
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "Amount": 15798,
    "Installments": 1,
    "Interest": "ByMerchant",
    "ProductId": 1,
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "SecurityCodeStatus": "Nonexistent",
      "BrandId": 1,
      "IssuerId": 2,
      "InputMode": "MagStripe",
      "AuthenticationMethod": "OnlinePassword",
      "TrackOneData": "A1234567890123456^FULANO OLIVEIRA SA ^12345678901234567890123",
      "TrackTwoData": "0123456789012345=012345678901234",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModuleAndContactless",
      "ReturnDataInfo": "00"
    }
  }
}
```

#### Resposta

```json
{
  "MerchantOrderId": "20180204",
  "Customer": {
    "Name": "[Guest]"
  },
  "Payment": {
    "Installments": 1,
    "Interest": "ByMerchant",
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "TruncateCardNumberWhenPrinting": true,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlineAuthentication",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "ServiceTaxAmount": 0,
    "SoftDescriptor": "Description",
    "ProductId": 1,
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    },
    "Amount": 15798,
    "ReceivedDate": "2019-04-15T12:00:00Z",
    "CapturedAmount": 15798,
    "Provider": "Cielo",
    "ConfirmationStatus": 0,
    "InitializationVersion": 1558708320029,
    "EmvResponseData": "123456789ABCD1345DEA",
    "Status": 2,
    "IsSplitted": false,
    "ReturnCode": 0,
    "ReturnMessage": "Successful",
    "PaymentId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
    "Type": "PhysicalDebitCard",
    "Currency": "BRL",
    "Country": "BRA",
    "Links": [
      {
        "Method": "GET",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "DELETE",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "PUT",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/confirmation"
      }
    ],
    "PrintMessage": [
      {
        "Position": "Top",
        "Message": "Transação autorizada"
      },
      {
        "Position": "Bottom",
        "Message": "Obrigado e volte sempre!"
      }
    ]
  }
}
```

### Venda com cartão de débito com leitura de tarja e senha

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/</span></aside>

```json
{
  "MerchantOrderId": "201904150003",
  "Payment": {
    "Type": "PhysicalDebitCard",
    "SoftDescriptor": "Description",
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "Amount": 15798,
    "ProductId": 1,
    "DebitCard": {
      "ExpirationDate": "12/2020",
      "SecurityCodeStatus": "Nonexistent",
      "BrandId": 1,
      "IssuerId": 2,
      "InputMode": "MagStripe",
      "AuthenticationMethod": "OnlinePassword",
      "TrackOneData": "A1234567890123456^FULANO OLIVEIRA SA ^12345678901234567890123",
      "TrackTwoData": "0123456789012345=012345678901234",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    }
  }
}
```

#### Resposta

```json
{
  "MerchantOrderId": "20180204",
  "Customer": {
    "Name": "[Guest]"
  },
  "Payment": {
    "Installments": 1,
    "Interest": "ByMerchant",
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "TruncateCardNumberWhenPrinting": true,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlineAuthentication",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "ServiceTaxAmount": 0,
    "SoftDescriptor": "Description",
    "ProductId": 1,
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    },
    "Amount": 15798,
    "ReceivedDate": "2019-04-15T12:00:00Z",
    "CapturedAmount": 15798,
    "Provider": "Cielo",
    "ConfirmationStatus": 0,
    "InitializationVersion": 1558708320029,
    "EmvResponseData": "123456789ABCD1345DEA",
    "Status": 2,
    "IsSplitted": false,
    "ReturnCode": 0,
    "ReturnMessage": "Successful",
    "PaymentId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
    "Type": "PhysicalDebitCard",
    "Currency": "BRL",
    "Country": "BRA",
    "Links": [
      {
        "Method": "GET",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "DELETE",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "PUT",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/confirmation"
      }
    ],
    "PrintMessage": [
      {
        "Position": "Top",
        "Message": "Transação autorizada"
      },
      {
        "Position": "Bottom",
        "Message": "Obrigado e volte sempre!"
      }
    ]
  }
}
```

### Venda com cartão de crédito com EMV com senha online

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/</span></aside>

```json
{
  "MerchantOrderId": "201904150004",
  "Payment": {
    "Type": "PhysicalCreditCard",
    "SoftDescriptor": "Description",
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "Amount": 15798,
    "Installments": 1,
    "Interest": "ByMerchant",
    "ProductId": 1,
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlinePassword",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    }
  }
}
```

#### Resposta

```json
{
  "MerchantOrderId": "20180204",
  "Customer": {
    "Name": "[Guest]"
  },
  "Payment": {
    "Installments": 1,
    "Interest": "ByMerchant",
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "TruncateCardNumberWhenPrinting": true,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlineAuthentication",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "ServiceTaxAmount": 0,
    "SoftDescriptor": "Description",
    "ProductId": 1,
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    },
    "Amount": 15798,
    "ReceivedDate": "2019-04-15T12:00:00Z",
    "CapturedAmount": 15798,
    "Provider": "Cielo",
    "ConfirmationStatus": 0,
    "InitializationVersion": 1558708320029,
    "EmvResponseData": "123456789ABCD1345DEA",
    "Status": 2,
    "IsSplitted": false,
    "ReturnCode": 0,
    "ReturnMessage": "Successful",
    "PaymentId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
    "Type": "PhysicalDebitCard",
    "Currency": "BRL",
    "Country": "BRA",
    "Links": [
      {
        "Method": "GET",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "DELETE",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "PUT",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/confirmation"
      }
    ],
    "PrintMessage": [
      {
        "Position": "Top",
        "Message": "Transação autorizada"
      },
      {
        "Position": "Bottom",
        "Message": "Obrigado e volte sempre!"
      }
    ]
  }
}
```

### Venda com cartão de débito com EMV e senha online

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/</span></aside>

```json
{
  "MerchantOrderId": "201904150005",
  "Payment": {
    "Type": "PhysicalDebitCard",
    "SoftDescriptor": "Description",
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "Amount": 15798,
    "ProductId": 1,
    "DebitCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlinePassword",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    }
  }
}
```

#### Resposta

```json
{
  "MerchantOrderId": "20180204",
  "Customer": {
    "Name": "[Guest]"
  },
  "Payment": {
    "Installments": 1,
    "Interest": "ByMerchant",
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "TruncateCardNumberWhenPrinting": true,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlineAuthentication",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "ServiceTaxAmount": 0,
    "SoftDescriptor": "Description",
    "ProductId": 1,
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    },
    "Amount": 15798,
    "ReceivedDate": "2019-04-15T12:00:00Z",
    "CapturedAmount": 15798,
    "Provider": "Cielo",
    "ConfirmationStatus": 0,
    "InitializationVersion": 1558708320029,
    "EmvResponseData": "123456789ABCD1345DEA",
    "Status": 2,
    "IsSplitted": false,
    "ReturnCode": 0,
    "ReturnMessage": "Successful",
    "PaymentId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
    "Type": "PhysicalDebitCard",
    "Currency": "BRL",
    "Country": "BRA",
    "Links": [
      {
        "Method": "GET",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "DELETE",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "PUT",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/confirmation"
      }
    ],
    "PrintMessage": [
      {
        "Position": "Top",
        "Message": "Transação autorizada"
      },
      {
        "Position": "Bottom",
        "Message": "Obrigado e volte sempre!"
      }
    ]
  }
}
```

### Venda com vale alimentação (cartão de voucher) com EMV e senha online

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/</span></aside>

```json
{
  "MerchantOrderId": "201904150005",
  "Payment": {
    "Type": "PhysicalVoucherCard",
    "SoftDescriptor": "Description",
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "Amount": 15798,
    "ProductId": 1,
    "VoucherCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlinePassword",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    }
  }
}
```

#### Resposta

```json
{
  "MerchantOrderId": "20180204",
  "Customer": {
    "Name": "[Guest]"
  },
  "Payment": {
    "Installments": 1,
    "Interest": "ByMerchant",
    "CreditCard": {
      "ExpirationDate": "12/2020",
      "BrandId": 1,
      "IssuerId": 2,
      "TruncateCardNumberWhenPrinting": true,
      "InputMode": "Emv",
      "AuthenticationMethod": "OnlineAuthentication",
      "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
      "PinBlock": {
        "EncryptedPinBlock": "2280F6BDFD0C038D",
        "EncryptionType": "Dukpt3Des",
        "KsnIdentification": "1231vg31fv231313123"
      }
    },
    "PaymentDateTime": "2019-04-15T12:00:00Z",
    "ServiceTaxAmount": 0,
    "SoftDescriptor": "Description",
    "ProductId": 1,
    "PinPadInformation": {
      "TerminalId": "10000001",
      "SerialNumber": "ABC123",
      "PhysicalCharacteristics": "PinPadWithChipReaderWithSamModule",
      "ReturnDataInfo": "00"
    },
    "Amount": 15798,
    "ReceivedDate": "2019-04-15T12:00:00Z",
    "CapturedAmount": 15798,
    "Provider": "Cielo",
    "ConfirmationStatus": 0,
    "InitializationVersion": 1558708320029,
    "EmvResponseData": "123456789ABCD1345DEA",
    "Status": 2,
    "IsSplitted": false,
    "ReturnCode": 0,
    "ReturnMessage": "Successful",
    "PaymentId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
    "Type": "PhysicalDebitCard",
    "Currency": "BRL",
    "Country": "BRA",
    "Links": [
      {
        "Method": "GET",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "DELETE",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
      },
      {
        "Method": "PUT",
        "Rel": "self",
        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/confirmation"
      }
    ],
    "PrintMessage": [
      {
        "Position": "Top",
        "Message": "Transação autorizada"
      },
      {
        "Position": "Bottom",
        "Message": "Obrigado e volte sempre!"
      }
    ]
  }
}
```

# Confirmação

## Confirma um pagamento

Quando o pagamento retornar sucesso e pode ser confirmado.

Esta operação requer o PaymentId recebido no retorno do pagamento, além dos dados EmvData se o pagamento foi realizado atráves de Chip.

A confirmação somente é necessária para pagamentos feitos através do POS.

### Confirmação de pagamento usando cartão digitado

#### Requisição

<aside class="request"><span class="method put">PUT</span> <span class="endpoint">/1/physicalSales/{PaymentId}/confirmation</span></aside>

```json
null
```

#### Resposta

```json
{
  "ConfirmationStatus": 1,
  "Status": 2,
  "ReturnCode": 0,
  "ReturnMessage": "Successful",
  "Links": [
    {
      "Method": "GET",
      "Rel": "self",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
    },
    {
      "Method": "POST",
      "Rel": "void",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids"
    }
  ]
}
```

### Confirmação de pagamento usando cartão EMV

#### Requisição

<aside class="request"><span class="method put">PUT</span> <span class="endpoint">/1/physicalSales/{PaymentId}/confirmation</span></aside>

```json
{
  "EmvData": "112233445566778899011AABBC012D3456789E0123FF45678AB901234C5D112233445566778800",
  "IssuerScriptResults": "0000"
}
```

#### Resposta

```json
{
  "ConfirmationStatus": 1,
  "Status": 2,
  "ReturnCode": 0,
  "ReturnMessage": "Successful",
  "Links": [
    {
      "Method": "GET",
      "Rel": "self",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
    },
    {
      "Method": "POST",
      "Rel": "void",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids"
    }
  ]
}
```

# Cancelamento

## Cancela um pagamento

### Cancelamento de pagamento com cartão digitado

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/{PaymentId}/voids/</span></aside>

```json
{
  "MerchantVoidId": 2019042204,
  "MerchantVoidDate": "2019-04-15T12:00:00Z",
  "Card": {
    "InputMode": "Typed",
    "CardNumber": 1234567812345678
  }
}
```

#### Resposta

```json
{
  "VoidId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
  "Status": 10,
  "ReturnCode": 0,
  "Links": [
    {
      "Method": "GET",
      "Rel": "self",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
    },
    {
      "Method": "POST",
      "Rel": "void",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids"
    },
    {
      "Method": "DELETE",
      "Rel": "reverse",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids/e5c889ea-5719-4e1a-a2da-f4f50d5bd7ca"
    },
    {
      "Method": "PUT",
      "Rel": "confirm",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids/e5c889ea-5719-4e1a-a2da-f4f50d5bd7ca/confirmation"
    }
  ]
}
```

### Cancelamento de pagamento com cartão presente

#### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/physicalSales/{PaymentId}/voids/</span></aside>

```json
{
  "MerchantVoidId": 2019042204,
  "MerchantVoidDate": "2019-04-15T12:00:00Z",
  "Card": {
    "InputMode": "MagStripe",
    "TrackOneData": "A1234567890123456^FULANO OLIVEIRA SA ^12345678901234567890123",
    "TrackTwoData": "0123456789012345=012345678901234"
  }
}
```

#### Resposta

```json
{
  "VoidId": "f15889ea-5719-4e1a-a2da-f4e50d5bd702",
  "Status": 10,
  "ReturnCode": 0,
  "Links": [
    {
      "Method": "GET",
      "Rel": "self",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
    },
    {
      "Method": "POST",
      "Rel": "void",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids"
    },
    {
      "Method": "DELETE",
      "Rel": "reverse",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids/e5c889ea-5719-4e1a-a2da-f4f50d5bd7ca"
    },
    {
      "Method": "PUT",
      "Rel": "confirm",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702/voids/e5c889ea-5719-4e1a-a2da-f4f50d5bd7ca/confirmation"
    }
  ]
}
```

# Desfaziamento

## Desfaz um pagamento

O pagamente retornou com sucesso e pode ser desfeito.

Deve-se solicitar o desfazimento através do PaymentId recebido no retorno do pagamento. 

### Requisição

<aside class="request"><span class="method delete">DELETE</span> <span class="endpoint">/1/physicalSales/{PaymentId}</span></aside>

```json
{
  "ConfirmationStatus": 2,
  "Status": 2,
  "ReturnCode": 0,
  "Links": [
    {
      "Method": "GET",
      "Rel": "self",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
    }
  ]
}
```

## Desfaz um pagamento

Quando o pagamente não retornar, o mesmo deve ser desfeito.

Para solicitar o desfazimento é necessário informar o MerchantOrderId enviado no pagamento.

### Requisição

<aside class="request"><span class="method delete">DELETE</span> <span class="endpoint">/1/physicalSales/orderId/{MerchantOrderId}</span></aside>

```json
{
  "ConfirmationStatus": 2,
  "Status": 2,
  "ReturnCode": 0,
  "Links": [
    {
      "Method": "GET",
      "Rel": "self",
      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/f15889ea-5719-4e1a-a2da-f4e50d5bd702"
    }
  ]
}
```
