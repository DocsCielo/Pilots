---
layout: manual
title: Manual de Integração Solução Omni 2
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

## Swagger Integração do mundo físico na API 3.0

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "Cielo 3.0",
    "description": "Integração do mundo físico na API 3.0",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "https://apisandbox.cieloecommerce.cielo.com.br/1",
      "description": "Sandbox"
    },
    {
      "url": "https://api.cieloecommerce.cielo.com.br/1",
      "description": "Produção"
    }
  ],
  "tags": [
    {
      "name": "Pagamento",
      "description": "Cria um novo pagamento"
    },
    {
      "name": "Desfazimento",
      "description": "Desfaz um pagamento"
    },
    {
      "name": "Confirmação",
      "description": "Confirma um pagamento"
    },
    {
      "name": "Cancelamento",
      "description": "Cancela um pagamento"
    }
  ],
  "paths": {
    "/physicalSales": {
      "post": {
        "tags": [
          "Pagamento"
        ],
        "summary": "Cria um novo pagamento",
        "description": "Quando um pagamento é criado (201 - Created), deve-se analisar o Status (Payment.Status) na resposta para certificar-se que o pagamento foi gerado com sucesso ou se houve alguma falha.",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/OrderRequest"
              },
              "examples": {
                "Venda com cartão de crédito digitado e sem senha": {
                  "value": {
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
                        "CardNumber": "0000000000000001",
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
                },
                "Venda com cartão de crédito com leitura de tarja e senha": {
                  "value": {
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
                },
                "Venda com cartão de débito com leitura de tarja e senha": {
                  "value": {
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
                },
                "Venda com cartão de crédito com EMV com senha online": {
                  "value": {
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
                        "EmvData": "950580400080009F37045EEBDE309F2701409F2608EE88439FC455604F9F100706010C03608000",
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
                },
                "Venda com cartão de débito com EMV e senha online": {
                  "value": {
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
                        "EmvData": "950580400080009F37045EEBDE309F2701409F2608EE88439FC455604F9F100706010C03608000",
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
                },
                "Venda com vale alimentação (cartão de voucher) com EMV e senha online": {
                  "value": {
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
                        "EmvData": "950580400080009F37045EEBDE309F2701409F2608EE88439FC455604F9F100706010C03608000",
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
                }
              }
            }
          }
        },
        "responses": {
          "201": {
            "description": "Resposta de sucesso na criação a venda",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/OrderResponse"
                },
                "example": {
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
                      "EmvData": "950580400080009F37045EEBDE309F2701409F2608EE88439FC455604F9F100706010C03608000",
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
                    "Status": 2,
                    "IsSplitted": false,
                    "ReturnCode": 0,
                    "ReturnMessage": "Successful",
                    "PaymentId": "00000000-0000-0000-0000-0000000000000",
                    "Type": "PhysicalDebitCard",
                    "Currency": "BRL",
                    "Country": "BRA",
                    "Links": [
                      {
                        "Method": "GET",
                        "Rel": "self",
                        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000"
                      },
                      {
                        "Method": "DELETE",
                        "Rel": "self",
                        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000"
                      },
                      {
                        "Method": "PUT",
                        "Rel": "self",
                        "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000/confirmation"
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
              }
            }
          },
          "400": {
            "$ref": "#/components/responses/400"
          },
          "401": {
            "$ref": "#/components/responses/401"
          },
          "403": {
            "$ref": "#/components/responses/403"
          }
        }
      }
    },
    "/physicalSales/{PaymentId}": {
      "delete": {
        "tags": [
          "Desfazimento"
        ],
        "summary": "Desfaz um pagamento",
        "description": "O pagamente retornou com sucesso e pode ser desfeito.\n<br>Deve-se solicitar o desfazimento através do PaymentId recebido no retorno do pagamento.\n",
        "parameters": [
          {
            "name": "PaymentId",
            "in": "path",
            "required": true,
            "description": "Código do Pagamento",
            "schema": {
              "type": "string",
              "format": "uuid",
              "example": "00000000-0000-0000-0000-0000000000000"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "O pagamento foi desfeito com sucesso",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/PaymentBaseResponse"
                },
                "example": {
                  "ConfirmationStatus": 2,
                  "Status": 2,
                  "ReturnCode": 0,
                  "Links": [
                    {
                      "Method": "GET",
                      "Rel": "self",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000"
                    }
                  ]
                }
              }
            }
          },
          "401": {
            "$ref": "#/components/responses/401"
          },
          "403": {
            "$ref": "#/components/responses/403"
          },
          "404": {
            "description": "O pagamento informado não existe ou já foi desfeito"
          }
        }
      }
    },
    "/physicalSales/{PaymentId}/confirmation": {
      "put": {
        "tags": [
          "Confirmação"
        ],
        "summary": "Confirma um pagamento",
        "description": "Quando o pagamento retornar sucesso e pode ser confirmado.\n<br>Esta operação requer o PaymentId recebido no retorno do pagamento, além dos dados EmvData se o pagamento foi realizado atráves de Chip.\n<br>A confirmação somente é necessária para pagamentos feitos através do POS.\n",
        "parameters": [
          {
            "name": "PaymentId",
            "in": "path",
            "required": true,
            "description": "Código do Pagamento",
            "schema": {
              "type": "string",
              "format": "uuid"
            }
          }
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "EmvData": {
                    "type": "string",
                    "description": "Dados da transação EMV<br>Dados obtidos através do comando PP_GoOnChip na BC\n",
                    "example": "950580400080009F37045EEBDE309F2701409F2608EE88439FC455604F9F100706010C03608000"
                  },
                  "IssuerScriptResults": {
                    "type": "string",
                    "description": "Resultado dos scripts EMV do emissor",
                    "example": "0000"
                  }
                }
              },
              "examples": {
                "Confirmação de pagamento usando cartão digitado": {
                  "value": null
                },
                "Confirmação de pagamento usando cartão EMV": {
                  "value": {
                    "EmvData": "950580400080009F37045EEBDE309F2701409F2608EE88439FC455604F9F100706010C03608000",
                    "IssuerScriptResults": "0000"
                  }
                }
              }
            }
          }
        },
        "responses": {
          "200": {
            "description": "A confirmação foi feita com sucesso",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/PaymentBaseResponse"
                },
                "example": {
                  "ConfirmationStatus": 1,
                  "Status": 2,
                  "ReturnCode": 0,
                  "ReturnMessage": "Successful",
                  "Links": [
                    {
                      "Method": "GET",
                      "Rel": "self",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000"
                    },
                    {
                      "Method": "POST",
                      "Rel": "void",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000/voids"
                    }
                  ]
                }
              }
            }
          },
          "401": {
            "$ref": "#/components/responses/401"
          },
          "403": {
            "$ref": "#/components/responses/403"
          },
          "404": {
            "description": "O pagamento não existe ou foi desfeito"
          }
        }
      }
    },
    "/physicalSales/{PaymentId}/voids": {
      "post": {
        "tags": [
          "Cancelamento"
        ],
        "summary": "Cancela um pagamento",
        "parameters": [
          {
            "name": "PaymentId",
            "in": "path",
            "required": true,
            "description": "Código do Pagamento",
            "schema": {
              "type": "string",
              "format": "uuid",
              "example": "00000000-0000-0000-0000-0000000000000"
            }
          },
          {
            "name": "RequestId",
            "in": "header",
            "description": "Identificador do Request,\nutilizado quando o lojista usa diferentes servidores para cada GET/POST/PUT\n",
            "schema": {
              "type": "string",
              "format": "uuid",
              "example": "00000000-0000-0000-0000-0000000000000"
            }
          }
        ],
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/VoidRequest"
              },
              "examples": {
                "Cancelamento de pagamento com cartão digitado": {
                  "value": {
                    "MerchantVoidId": 2019042204,
                    "MerchantVoidDate": "2019-04-15T12:00:00Z",
                    "Card": {
                      "InputMode": "Typed",
                      "CardNumber": "5488260250125846"
                    }
                  }
                },
                "Cancelamento de pagamento com cartão presente": {
                  "value": {
                    "MerchantVoidId": 2019042204,
                    "MerchantVoidDate": "2019-04-15T12:00:00Z",
                    "Card": {
                      "InputMode": "MagStripe",
                      "TrackOneData": "B3764 361234 56006^NOME NOME NOME NOME NOME N^0905060640431",
                      "TrackTwoData": "6002938264523821=09050606404312376450"
                    }
                  }
                }
              }
            }
          }
        },
        "responses": {
          "201": {
            "description": "Resposta de sucesso no cancelamento",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/VoidResponse"
                },
                "example": {
                  "VoidId": "00000000-0000-0000-0000-0000000000000",
                  "Status": 10,
                  "ReturnCode": 0,
                  "Links": [
                    {
                      "Method": "GET",
                      "Rel": "self",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000"
                    },
                    {
                      "Method": "POST",
                      "Rel": "void",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000/voids"
                    },
                    {
                      "Method": "DELETE",
                      "Rel": "reverse",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000/voids/e5c889ea-5719-4e1a-a2da-f4f50d5bd7ca"
                    },
                    {
                      "Method": "PUT",
                      "Rel": "confirm",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000/voids/e5c889ea-5719-4e1a-a2da-f4f50d5bd7ca/confirmation"
                    }
                  ]
                }
              }
            }
          },
          "400": {
            "$ref": "#/components/responses/400"
          },
          "401": {
            "$ref": "#/components/responses/401"
          },
          "403": {
            "$ref": "#/components/responses/403"
          }
        }
      }
    },
    "/physicalSales/orderId/{MerchantOrderId}": {
      "delete": {
        "tags": [
          "Desfazimento"
        ],
        "summary": "Desfaz um pagamento",
        "description": "Quando o pagamente não retornar, o mesmo deve ser desfeito.\n<br>Para solicitar o desfazimento é necessário informar o MerchantOrderId enviado no pagamento.\n",
        "parameters": [
          {
            "name": "MerchantOrderId",
            "in": "path",
            "required": true,
            "description": "Número do documento gerado automáticamente pelo terminal e\nincrementado de 1 acada transação realizada no terminal\n",
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "O pagamento foi desfeito com sucesso",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/PaymentBaseResponse"
                },
                "example": {
                  "ConfirmationStatus": 2,
                  "Status": 2,
                  "ReturnCode": 0,
                  "Links": [
                    {
                      "Method": "GET",
                      "Rel": "self",
                      "Href": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000"
                    }
                  ]
                }
              }
            }
          },
          "401": {
            "$ref": "#/components/responses/401"
          },
          "403": {
            "$ref": "#/components/responses/403"
          },
          "404": {
            "description": "O pagamento informado não existe ou já foi desfeito"
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "BadRequestError": {
        "type": "object",
        "properties": {
          "Code": {
            "type": "integer",
            "example": 126,
            "description": "Código de erro\n"
          },
          "Message": {
            "type": "string",
            "example": "The credit card expiration date is in an invalid format",
            "description": "Mensagem de erro\n"
          }
        }
      },
      "BrandInformation": {
        "type": "object",
        "properties": {
          "Type": {
            "type": "string",
            "example": "Mastercard"
          },
          "Description": {
            "type": "string",
            "example": "Crédito Nubank"
          }
        }
      },
      "CardRequest": {
        "type": "object",
        "required": [
          "ExpirationDate",
          "BrandId",
          "IssuerId",
          "InputMode",
          "AuthenticationMethod"
        ],
        "properties": {
          "CardNumber": {
            "type": "string",
            "maxLength": 19,
            "example": "4572370204544658",
            "description": "Número do cartão (PAN)\n"
          },
          "ExpirationDate": {
            "type": "string",
            "format": "MM/yyyy",
            "example": "12/2021",
            "description": "Data de validade do cartão\n<br>Dado obtido através do comando PP_GetCard na BC no momento da captura da transação\n"
          },
          "SecurityCodeStatus": {
            "type": "string",
            "enum": [
              "Collected",
              "Unreadable",
              "Nonexistent"
            ],
            "description": "Status da coleta de código de segurança (CVV)\n"
          },
          "SecurityCode": {
            "type": "string",
            "minLength": 3,
            "maxLength": 4,
            "example": "123",
            "description": "Código de segurança (CVV)\n"
          },
          "BrandId": {
            "type": "integer",
            "description": "Identificação da bandeira obtida através do campo BrandId da PRODUCT TABLE.\n"
          },
          "IssuerId": {
            "type": "integer",
            "description": "Código do emissor obtido através do campo IssuerId da BIN TABLE.\n"
          },
          "InputMode": {
            "type": "string",
            "enum": [
              "Typed",
              "MagStripe",
              "Emv"
            ],
            "description": "Identificação do modo de captura do cartão na transação. Essa informação deve ser obtida através do retorno da função PP_GetCard da BC.\n<br>“00” – Magnético \n<br>“01” - Moedeiro VISA Cash sobre TIBC v1 \n<br>“02” - Moedeiro VISA Cash sobre TIBC v3 \n<br>“03” – EMV com contato \n<br>“04” - Easy-Entry sobre TIBC v1 \n<br>“05” - Chip sem contato simulando tarja “06” - EMV sem contato.\n"
          },
          "TrackOneData": {
            "type": "string",
            "example": "B3764361234006^NOME NOME NOME NOME NOME N^0905060640431",
            "description": "Dados da trilha 1\n<br>Obtidos através do comando PP_GetCard na BC no momento da captura da transação\n"
          },
          "TrackTwoData": {
            "type": "string",
            "example": "6002938264523821=09050606404312376450",
            "description": "Dados da trilha 2\n<br>Obtidos através do comando PP_GetCard na BC no momento da captura da transação\n"
          },
          "EmvData": {
            "type": "string",
            "example": "950580400080009F37045EEBDE309F2701409F2608EE88439FC455604F9F100706010C03608000",
            "description": "Dados da transação EMV\n<br>Obtidos através do comando PP_GoOnChip na BC\n"
          },
          "TruncateCardNumberWhenPrinting": {
            "type": "boolean",
            "description": "Indica se o número do cartão será truncado no momento da impressão do comprovante. A solução de captura deve tomar essa decisão com base no “confParamOp03” presente nas tabelas BIN TABLE, PARAMETER TABLE e ISSUER TABLE.\n"
          },
          "AuthenticationMethod": {
            "type": "string",
            "enum": [
              "NoPassword",
              "OnlineAuthentication",
              "OfflineAuthentication"
            ],
            "description": "Método de autenticação\n<br>-  Se o cartão foi lido a partir da digitação verificar o bit 3 do atributo confParamOp04 das tabelas binTable, parameterTable e issuerTable. Se todos estiverem habilitados, a senha deve ser capturada e o authenticationMethod assume valor 2. Caso contrário, assume valor 1;\n<br>-  Se o cartão foi lido a partir da trilha verificar o bit 3 do atributo confParamOp04 das tabelas binTable, parameterTable e issuerTable. Se todos estiverem habilitados, deve ser verificado o bit 2 do mesmo campo. Se este estiver com valor 1 deve ser capturada a senha. Se estiver com valor 0 a captura da senha vai depender do último dígito do service code;\n<br>-  Se o cartão foi lido através do chip EMV, o authenticationMethod será preenchido com base no retorno da função PP_GoOnChip(). No resultado PP_GoOnChip(), onde se o campo da posição 003 do retorno da PP_GoOnChip() estiver com valor 1 indica que o pin foi validado off-line, o authenticationMethod será 3. Se o campo da posição 003 e o campo da posição 006 do retorno da PP_GoOnChip() estiverem com valor 0, o authenticationMethod será 1. Se o campo da posição 003 e o campo da posição 006 do retorno da PP_GoOnChip() estiverem com valores 0 e 1 respectivamente, o authenticationMethod será 2.\n<br>1  - Sem senha = “NoPassword”;\n<br>2  - Senha online = “Online Authentication”;\n<br>3  - Senha off-line = “Offline Authentication”.\n"
          },
          "PinBlock": {
            "$ref": "#/components/schemas/PinBlockInformation"
          }
        }
      },
      "CardResponse": {
        "allOf": [
          {
            "$ref": "#/components/schemas/CardRequest"
          },
          {
            "type": "object",
            "properties": {
              "BrandInformation": {
                "$ref": "#/components/schemas/BrandInformation"
              },
              "FallbackIndicator": {
                "type": "boolean"
              }
            }
          }
        ]
      },
      "Link": {
        "type": "object",
        "properties": {
          "Method": {
            "type": "string",
            "enum": [
              "POST",
              "GET",
              "PUT"
            ],
            "description": "Método HTTP a ser utilizado na operação\n"
          },
          "Rel": {
            "type": "string",
            "enum": [
              "self",
              "cancel",
              "confirm"
            ],
            "description": "Referência da operação\n"
          },
          "Href": {
            "type": "string",
            "example": "https://api.cieloecommerce.cielo.com.br/1/physicalSales/00000000-0000-0000-0000-0000000000000",
            "description": "Endereço de URL de chamada da API\n"
          }
        }
      },
      "OrderBase": {
        "type": "object",
        "required": [
          "MerchantOrderId",
          "Payment"
        ],
        "properties": {
          "MerchantOrderId": {
            "type": "string",
            "description": "Número do documento gerado automáticamente pelo terminal e incrementado de 1 acada transação realizada no terminal\n"
          }
        }
      },
      "OrderRequest": {
        "allOf": [
          {
            "$ref": "#/components/schemas/OrderBase"
          },
          {
            "type": "object",
            "required": [
              "Payment"
            ],
            "properties": {
              "Payment": {
                "oneOf": [
                  {
                    "$ref": "#/components/schemas/PhysicalCreditCardPaymentRequest"
                  },
                  {
                    "$ref": "#/components/schemas/PhysicalDebitCardPaymentRequest"
                  },
                  {
                    "$ref": "#/components/schemas/PhysicalVoucherCardPaymentRequest"
                  }
                ]
              }
            }
          }
        ]
      },
      "OrderResponse": {
        "allOf": [
          {
            "$ref": "#/components/schemas/OrderBase"
          },
          {
            "type": "object",
            "properties": {
              "Customer": {
                "type": "object",
                "properties": {
                  "Name": {
                    "type": "string",
                    "description": "Nome do portador obtido através da leitura da trilha 1 do cartão utilizado na transação.\n"
                  }
                }
              }
            }
          },
          {
            "type": "object",
            "required": [
              "Payment"
            ],
            "properties": {
              "Payment": {
                "oneOf": [
                  {
                    "$ref": "#/components/schemas/PhysicalCreditCardPaymentResponse"
                  },
                  {
                    "$ref": "#/components/schemas/PhysicalDebitCardPaymentResponse"
                  },
                  {
                    "$ref": "#/components/schemas/PhysicalVoucherCardPaymentResponse"
                  }
                ]
              }
            }
          }
        ]
      },
      "PaymentBaseResponse": {
        "allOf": [
          {
            "type": "object",
            "properties": {
              "ConfirmationStatus": {
                "type": "integer",
                "format": "int16",
                "description": "Status da confirmação.\n  <br>0 = Pendente\n  <br>1 = Confirmado\n  <br>2 = Desfeito\n"
              }
            }
          },
          {
            "$ref": "#/components/schemas/ResponseBase"
          }
        ]
      },
      "PaymentRequest": {
        "type": "object",
        "required": [
          "Amount",
          "PaymentDateTime",
          "PinPadInformation",
          "ProductId"
        ],
        "properties": {
          "Amount": {
            "type": "integer",
            "format": "int64",
            "description": "Valor da transação (1079 = R$10,79)\n"
          },
          "Currency": {
            "type": "string",
            "enum": [
              "BRL"
            ],
            "default": "BRL",
            "description": "Moeda (Preencher com “BRL”)\n"
          },
          "Country": {
            "type": "string",
            "enum": [
              "BRA"
            ],
            "default": "BRA",
            "description": "País (Preencher com “BRA”)\n"
          },
          "SoftDescriptor": {
            "type": "string",
            "maxLength": 13,
            "description": "Identificação do estabelecimento (nome reduzido) a ser impresso e identificado na fatura\n"
          },
          "PaymentDateTime": {
            "type": "string",
            "format": "date-time",
            "example": "2019-04-15T12:00:00Z",
            "description": "Data e Hora da captura da transação\n"
          },
          "PinPadInformation": {
            "$ref": "#/components/schemas/PinPadInformation"
          },
          "ProductId": {
            "type": "integer",
            "description": "Código do produto identificado através do bin do cartão.\n"
          },
          "SubordinatedMerchantId": {
            "type": "string",
            "format": "uuid",
            "example": "00000000-0000-0000-0000-0000000000000",
            "description": "Identificação do estabeleciomento\n"
          }
        }
      },
      "PaymentResponse": {
        "allOf": [
          {
            "$ref": "#/components/schemas/PaymentRequest"
          },
          {
            "$ref": "#/components/schemas/PaymentBaseResponse"
          },
          {
            "type": "object",
            "properties": {
              "IsSplitted": {
                "type": "boolean",
                "description": "Identifica se a transação foi feita com uso ou não do serviço de Split\n"
              },
              "PaymentId": {
                "type": "string",
                "format": "uuid",
                "example": "00000000-0000-0000-0000-0000000000000",
                "description": "Código do Pagamento\n"
              },
              "PrintMessage": {
                "type": "array",
                "items": {
                  "$ref": "#/components/schemas/PrintMessage"
                }
              },
              "Provider": {
                "type": "string",
                "description": "Nome do Adquirente\n"
              },
              "ServiceTaxAmount": {
                "type": "integer",
                "description": "Valor de taxa de serviço\n"
              },
              "ReceivedDate": {
                "type": "string",
                "format": "date-time",
                "description": "Data de recebimento\n"
              },
              "CapturedAmount": {
                "type": "integer",
                "description": "Valor capturado na transação\n"
              }
            }
          }
        ]
      },
      "PhysicalCreditCardPaymentBase": {
        "type": "object",
        "required": [
          "Type",
          "TerminalId",
          "CreditCard"
        ],
        "properties": {
          "Type": {
            "type": "string",
            "enum": [
              "PhysicalCreditCard"
            ],
            "description": "Tipo da Transação\n"
          },
          "Installments": {
            "type": "integer",
            "format": "byte",
            "default": 1,
            "example": 10,
            "description": "Quantidade de Parcelas\n<br>Varia de 2 a 99 para transação de financiamento. Deve ser verificado os atributos maxOfPayments1, maxOfPayments2, maxOfPayments3 e minValOfPayments da tabela productTable.\n"
          },
          "Interest": {
            "type": "string",
            "default": "ByMerchant",
            "enum": [
              "ByMerchant",
              "ByIssuer"
            ],
            "description": "Tipo de Parcelamento\n<br>-  Se o bit 6 do atributo confParamOp05, presente nas tabelas issuerTable e binTable e bit 6 do atributo confParamOp03 da tabela productTable estiverem todos habilitados indica que o tipo de parcelamento sem juros pode ser efetuado.\n<br>-  Se o bit 7 do atributo confParamOp05, presente nas tabelas issuerTable e binTable e bit 7 do atributo confParamOp03 da tabela productTable estiverem todos habilitados indica que o tipo de parcelamento com juros pode ser efetuado. Sem juros = “ByMerchant”; Com juros = “ByIssuer”.\n"
          }
        }
      },
      "PhysicalCreditCardPaymentRequest": {
        "allOf": [
          {
            "$ref": "#/components/schemas/PaymentRequest"
          },
          {
            "$ref": "#/components/schemas/PhysicalCreditCardPaymentBase"
          },
          {
            "type": "object",
            "properties": {
              "CreditCard": {
                "$ref": "#/components/schemas/CardRequest"
              }
            }
          }
        ]
      },
      "PhysicalCreditCardPaymentResponse": {
        "allOf": [
          {
            "$ref": "#/components/schemas/PaymentResponse"
          },
          {
            "$ref": "#/components/schemas/PhysicalCreditCardPaymentBase"
          },
          {
            "type": "object",
            "properties": {
              "CreditCard": {
                "$ref": "#/components/schemas/CardResponse"
              }
            }
          }
        ]
      },
      "PhysicalDebitCardPaymentBase": {
        "type": "object",
        "required": [
          "Type",
          "DebitCard"
        ],
        "properties": {
          "Type": {
            "type": "string",
            "enum": [
              "PhysicalDebitCard"
            ],
            "description": "Tipo da Transação\n"
          }
        }
      },
      "PhysicalDebitCardPaymentRequest": {
        "allOf": [
          {
            "$ref": "#/components/schemas/PaymentRequest"
          },
          {
            "$ref": "#/components/schemas/PhysicalDebitCardPaymentBase"
          },
          {
            "type": "object",
            "properties": {
              "DebitCard": {
                "$ref": "#/components/schemas/CardRequest"
              }
            }
          }
        ]
      },
      "PhysicalDebitCardPaymentResponse": {
        "allOf": [
          {
            "$ref": "#/components/schemas/PaymentResponse"
          },
          {
            "$ref": "#/components/schemas/PhysicalDebitCardPaymentBase"
          },
          {
            "type": "object",
            "properties": {
              "DebitCard": {
                "$ref": "#/components/schemas/CardResponse"
              }
            }
          }
        ]
      },
      "PhysicalVoucherCardPaymentBase": {
        "type": "object",
        "required": [
          "Type",
          "VoucherCard"
        ],
        "properties": {
          "Type": {
            "type": "string",
            "enum": [
              "PhysicalVoucherCard"
            ],
            "description": "Tipo da Transação\n"
          }
        }
      },
      "PhysicalVoucherCardPaymentRequest": {
        "allOf": [
          {
            "$ref": "#/components/schemas/PaymentRequest"
          },
          {
            "$ref": "#/components/schemas/PhysicalVoucherCardPaymentBase"
          },
          {
            "type": "object",
            "properties": {
              "VoucherCard": {
                "$ref": "#/components/schemas/CardRequest"
              }
            }
          }
        ]
      },
      "PhysicalVoucherCardPaymentResponse": {
        "allOf": [
          {
            "$ref": "#/components/schemas/PaymentResponse"
          },
          {
            "$ref": "#/components/schemas/PhysicalVoucherCardPaymentBase"
          },
          {
            "type": "object",
            "properties": {
              "VoucherCard": {
                "$ref": "#/components/schemas/CardResponse"
              }
            }
          }
        ]
      },
      "PinBlockInformation": {
        "type": "object",
        "description": "Processo de criptografia de senha",
        "required": [
          "EncryptedPinBlock",
          "EncryptionType",
          "KsnIdentification"
        ],
        "properties": {
          "EncryptedPinBlock": {
            "type": "string",
            "example": "2280F6BDFD0C038D",
            "description": "PINBlock Criptografado\n<br>- Para transações EMV, esse campo é obtido através do retorno da função PP_GoOnChip(), mais especificamente das posições 007 até a posição 022;\n<br>- Para transações digitadas e com tarja magnética, verificar as posições 001 até 016 do retorno da função PP_GetPin().\n"
          },
          "EncryptionType": {
            "type": "string",
            "enum": [
              "DukptDes",
              "Dukpt3Des",
              "MasterKey"
            ],
            "description": "Tipo de Criptografia\n"
          },
          "KsnIdentification": {
            "type": "string",
            "description": "Identificação do KSN\n<br>- Para transações EMV esse campo é obtido através do retorno da função PP_GoOnChip() nas posições 023 até 042; \n<br>- Para transações digitadas e com tarja magnética, verificar as posições 017 até 036 do retorno da função PP_GetPin(). \n"
          }
        }
      },
      "PinPadInformation": {
        "type": "object",
        "required": [
          "TerminalId",
          "SerialNumber",
          "PhysicalCharacteristics",
          "ReturnDataInfo"
        ],
        "properties": {
          "TerminalId": {
            "type": "string",
            "example": "A1B2C3",
            "description": "Número Lógico definido no Concentrador Cielo.\n"
          },
          "SerialNumber": {
            "type": "string",
            "example": 151819,
            "description": "Número de Série do Equipamento.\n"
          },
          "PhysicalCharacteristics": {
            "type": "string",
            "enum": [
              "WithoutPinPad",
              "PinPadWithoutChipReader",
              "PinPadWithChipReaderWithoutSamModule",
              "PinPadWithChipReaderWithSamModule",
              "NotCertifiedPinPad",
              "PinPadWithChipReaderWithoutSamAndContactless",
              "PinPadWithChipReaderWithSamModuleAndContactless"
            ],
            "description": "Sem PIN-pad = “WithoutPinPad”;\n<br>PIN-pad sem leitor de Chip = “PinpadWithoutChipReader”;\n<br>PIN-pad com leitor de Chip sem módulo SAM = “PinPadWithChipReaderWithoutSamModule”;\n<br>PIN-pad com leitor de Chip com módulo SAM = “PinPadWithChipReaderWithSamModule”;\n<br>PIN-pad não homologado = “NotCertifiedPinPad”;\n<br>PIN-pad com leitor de Chip sem SAM e Cartão Sem Contato = “PinpadWithChipReaderWithoutSamAndContactless”;\n<br>PIN-pad com leitor de Chip com SAM e Cartão Sem Contato = “PinpadWithChipReaderWithSamAndContactless”. \n<br>Obs. Caso a aplicação não consiga informar os dados acima, deve obter tais informações através do retorno da função PP_GetInfo() da BC.\n"
          },
          "ReturnDataInfo": {
            "type": "string",
            "description": "Retorno da função PP_GetInfo() da biblioteca compartilhada\n",
            "example": 0
          }
        }
      },
      "PrintMessage": {
        "type": "object",
        "properties": {
          "Position": {
            "type": "string",
            "enum": [
              "Top",
              "Middle",
              "Bottom"
            ],
            "default": "Top",
            "description": "Posição da mensagem no comprovante\n  * Top - Início do comprovante, antes do código do estabelecimento\n  * Middle - Meio do comprovante, após a descrição dos valores\n  * Bottom - Final do comprovante\n"
          },
          "Message": {
            "type": "string",
            "description": "Indica a mensagem que deve ser impressa no comprovante de acordo com a posição indicada no campo Position.\n"
          }
        }
      },
      "ResponseBase": {
        "type": "object",
        "properties": {
          "Status": {
            "type": "integer",
            "format": "int16",
            "description": "Status da transação\n  <br>0 = Não Finalizado\n  <br>1 = Autorizado\n  <br>2 = Pago\n  <br>3 = Negado\n  <br>10 = Cancelado\n  <br>13 = Abortado\n"
          },
          "ReasonCode": {
            "type": "integer",
            "format": "int16",
            "example": 0,
            "description": "Código de referência para análises\n"
          },
          "ReasonMessage": {
            "type": "string",
            "example": "Successful",
            "description": "Mensagem explicativa para análise\n"
          },
          "ReturnCode": {
            "type": "string",
            "example": "4",
            "description": "Código de erro/resposta da transação\n"
          },
          "ReturnMessage": {
            "type": "string",
            "example": "Transação Autorizada",
            "description": "Mensagem de erro/resposta da transação\n"
          },
          "Links": {
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/Link"
            }
          }
        }
      },
      "VoidRequest": {
        "type": "object",
        "required": [
          "MerchantVoidId",
          "MerchantVoidDate",
          "Card"
        ],
        "properties": {
          "MerchantVoidId": {
            "type": "string",
            "description": "Número do documento gerado automáticamente pelo terminal e incrementado de 1 acada transação realizada no terminal\n",
            "example": "2019042204"
          },
          "MerchantVoidDate": {
            "type": "string",
            "format": "date-time",
            "description": "Data do cancelamento.\n"
          },
          "Card": {
            "type": "object",
            "description": "Informações do cartão.",
            "properties": {
              "CardNumber": {
                "type": "string",
                "description": "Número do cartão\n<br>Requerido quando a transação for digitada.\n",
                "example": "5488260250125846"
              },
              "TrackOneData": {
                "type": "string",
                "description": "Dados da trilha 1\n<br>Dado obtido através do comando PP_GetCard na BC no momento da captura da transação\n",
                "example": "B3764 361234 56006^NOME NOME NOME NOME NOME N^0905060640431"
              },
              "TrackTwoData": {
                "type": "string",
                "description": "Dados da trilha 2\n<br>Dado obtido através do comando PP_GetCard na BC no momento da captura da transação\n",
                "example": "6002938264523821=09050606404312376450"
              }
            }
          }
        }
      },
      "VoidResponse": {
        "allOf": [
          {
            "type": "object",
            "properties": {
              "VoidId": {
                "type": "string",
                "format": "uuid",
                "example": "00000000-0000-0000-0000-0000000000000",
                "description": "Identificador do cancelamento\n"
              }
            }
          },
          {
            "$ref": "#/components/schemas/ResponseBase"
          }
        ]
      }
    },
    "responses": {
      "400": {
        "description": "Resposta contendo erros de contrato (Bad Request)\n",
        "content": {
          "application/json": {
            "schema": {
              "type": "array",
              "items": {
                "$ref": "#/components/schemas/BadRequestError"
              }
            }
          }
        }
      },
      "401": {
        "description": "Erro nas credenciais\n"
      },
      "403": {
        "description": "Operação não permitida\n"
      }
    },
    "headers": {
      "ExpiresAfter": {
        "schema": {
          "type": "string",
          "format": "date-time"
        },
        "description": "date in UTC when token expires\n"
      }
    },
    "securitySchemes": {
      "Autenticação": {
        "type": "oauth2",
        "description": "A autenticação é uma operação necessária para obtenção do token que será utilizado nas demais chamadas de APIs\n",
        "flows": {
          "clientCredentials": {
            "tokenUrl": "https://authsandbox.braspag.com.br/oauth2/token",
            "scopes": {
              "Administrator": "Admin everything",
              "AnalyticsApiOverview": "See the analytics",
              "AdminBackoffice": "Use the backoffice"
            }
          }
        }
      }
    }
  }
}
```

## Swagger Baixa de Parâmetros - Inicialização de terminal

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "Baixa de Parâmetros",
    "description": "Inicialização de terminal",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "https://parametersdownloadsandbox.cieloecommerce.cielo.com.br/api/v0.1",
      "description": "Sandbox"
    },
    {
      "url": "https://parametersdownload.cieloecommerce.cielo.com.br/api/v0.1",
      "description": "Produção"
    }
  ],
  "tags": [
    {
      "name": "Baixa de parâmetros",
      "description": "Essa operação é necessária para que o parceiro de negócio / Subadquirente receba todas as tabelas de parâmetros necessários para que a solução de captura possa efetuar as transações via chamada de API. Essa informação será recebida através de API e deverá ser instalada na BC\n"
    }
  ],
  "paths": {
    "/initialization/{TerminalId}": {
      "get": {
        "tags": [
          "Baixa de parâmetros"
        ],
        "summary": "Inicialização de um terminal",
        "description": "Solicita as tabelas e parametros para operação do terminal\n",
        "parameters": [
          {
            "name": "TerminalId",
            "in": "path",
            "required": true,
            "description": "Número Lógico definido no Concentrador Cielo\n",
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Resposta de sucesso no retorno de parâmetros",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/InitializeResponse"
                }
              }
            }
          },
          "401": {
            "$ref": "#/components/responses/401"
          },
          "403": {
            "$ref": "#/components/responses/403"
          },
          "404": {
            "description": "Terminal não encontrado"
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "Acquirer": {
        "type": "object",
        "properties": {
          "EnableContaclessCardReader": {
            "type": "boolean",
            "readOnly": true,
            "description": "Habilita Leitora Cartão Sem Contato\n"
          },
          "LockAppFunctionsExceptInitialization": {
            "type": "boolean",
            "readOnly": true,
            "description": "Bloquear as funções do aplicativo, com exceção da Inicialização\n"
          },
          "HasChipReader": {
            "type": "boolean",
            "readOnly": true,
            "description": "Indica que tem leitora de Chip-Card\n"
          },
          "HasMagneticTrackReader": {
            "type": "boolean",
            "readOnly": true,
            "description": "Indica que tem leitor da trilha magnética\n"
          },
          "HasKeyboard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Indica que tem teclado para digitação\n"
          }
        }
      },
      "Bin": {
        "type": "object",
        "properties": {
          "InitialBin": {
            "type": "string",
            "description": "Início do range de BIN’s.\n"
          },
          "FinalBin": {
            "type": "string",
            "description": "Final do range de BIN’s.\n"
          },
          "ProductId": {
            "format": "int32",
            "type": "integer",
            "description": "Chave estrangeira de “PRODUCT TABLE”.\n"
          },
          "Type": {
            "format": "int32",
            "type": "integer",
            "description": "Admite os seguintes valores\n<br>0 - ESPECÍFICO\n<br>1 – GENERICO.\n"
          },
          "AllowFallbackWhenChipReadingFails": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite fallback se houver erro na leitura do Chip.\n"
          },
          "AllowChargingMoedeiroFromCash": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite carga de moedeiro a partir de dinheiro em espécie em espécie.\n"
          },
          "AllowPurchaseWithCompreESaque": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite venda com Compre & Saque.\n"
          },
          "AllowOfflineFunctionExceptForEMVCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite função offline, exceto cartão EMV.\n"
          },
          "AllowTypingCardNumber": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite digitação do número do cartão.\n"
          },
          "MaskCardNumberUsingLast4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Imprimir apenas os 4 últimos dígitos do cartão.\n"
          },
          "MaskCardNumberUsingFirst6AndLas4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Imprimir os 6 primeiros e os 4 últimos dígitos do cartão.\n"
          },
          "AllowPrintCardHolderBalance": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite imprimir o saldo do portador.\n"
          },
          "AllowDisplayCardHolderBalance": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite exibir no display o saldo do portador.\n"
          },
          "AllowPrintingPartialCardNumberInReceipt": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite impressão parcial do número do cartão no comprovante das transações.\n"
          },
          "RestrictSaleWithDuplicateValueWhenPostdated": {
            "type": "boolean",
            "readOnly": true,
            "description": "Impede venda com valor duplicado para prédatamento.\n"
          },
          "RestrictSaleWithDuplicateValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Impede venda com valor duplicado.\n"
          },
          "RequiresPassword": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita senha.\n"
          },
          "InterpretsLastDigitOfSecurityCode": {
            "type": "boolean",
            "readOnly": true,
            "description": "Interpreta último dígito do Código de Serviço.\n"
          },
          "RequiresPasswordExceptForEMVCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita senha, exceto cartão EMV.\n"
          },
          "EnableAdditionalSecurityCodeOptions_Unreadable_NoCode": {
            "type": "boolean",
            "readOnly": true,
            "description": "Habilita opções “Ilegível” e “Não Possui” para Código de Segurança.\n"
          },
          "RequiresSecurityCodeWhenMagneticTrackIsRead": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita Código de Segurança na leitura de trilha.\n"
          },
          "RequiresSecurityCodeWhenCardNumberIsTyped": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita Código de Segurança para cartão digitado.\n"
          },
          "RequiresTypingLast4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita digitação dos últimos 4 dígitos.\n"
          },
          "AllowCaptureOfFirstInstallmentValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite captura do valor da primeira parcela.\n"
          },
          "AllowCaptureOfDownpaymentValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite captura do valor de entrada.\n"
          },
          "AllowGuaranteeHandling": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite tratamento de garantia.\n"
          },
          "AllowPostdatingTheFirstInstallmentForSaleAndCDCQuery": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite pré-datar a primeira parcela para venda e consulta CDC.\n"
          },
          "AllowPostdating": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite pré-datamento.\n"
          },
          "AllowCDCSale": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite venda CDC.\n"
          },
          "AllowFinancingByStore": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite financiamento pela Loja.\n"
          },
          "AllowFinancingByCreditCardCompany": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite financiamento pela Administradora.\n"
          },
          "ValidateCardTrack1": {
            "type": "boolean",
            "readOnly": true,
            "description": "Verifica Trilha 1 do cartão.\n"
          },
          "DoNotValidateCardModule10": {
            "type": "boolean",
            "readOnly": true,
            "description": "Não validar o Módulo 10 do cartão.\n"
          },
          "CheckExpiryDateWhenCardNumberIsTyped": {
            "type": "boolean",
            "readOnly": true,
            "description": "Verifica data de validade do cartão digitado.\n"
          },
          "CheckExpiryDateWhenMagneticTrackIsRead": {
            "type": "boolean",
            "readOnly": true,
            "description": "Verifica data de validade da trilha.\n"
          },
          "IssuerId": {
            "format": "int32",
            "type": "integer",
            "description": "Chave estrangeira de “ISSUER TABLE”.\n"
          }
        }
      },
      "Emv": {
        "type": "object",
        "properties": {
          "Aid": {
            "type": "string",
            "description": "Identificador da aplicação EMV.\n"
          },
          "TagsFirst": {
            "type": "string",
            "description": "Conjunto de tags obrigatórias enviadas o 1º Generate AC.\n"
          },
          "TagsSecond": {
            "type": "string",
            "description": "Conjunto de tags obrigatórias enviadas o 2º Generate AC.\n"
          },
          "IdxRecord": {
            "format": "int32",
            "type": "integer",
            "description": "-\n"
          },
          "Type": {
            "format": "int32",
            "type": "integer",
            "description": "Admite os seguintes valores\n<br>0 - CREDITO\n<br>1 – DEBITO\n"
          },
          "RCodeFirst": {
            "type": "string",
            "description": "-\n"
          },
          "RCodeSecond": {
            "type": "string",
            "description": "-\n"
          },
          "InvalidateFunctionIfCardIsOnBlacklist": {
            "type": "boolean",
            "readOnly": true,
            "description": "Invalida a função se o cartão consta na Lista Negra.\n"
          },
          "RequireBINToBeInCardRangeTable": {
            "type": "boolean",
            "readOnly": true,
            "description": "Obriga que o BIN esteja na tabela de Range de Cartões (tipo 2B).\n"
          },
          "StoreTransactionsRejectedByTerminalAppAndSendToHost": {
            "type": "boolean",
            "readOnly": true,
            "description": "Armazena e envia para o Host as transações rejeitadas pelo aplicativo do terminal.\n"
          },
          "NatEmvConctactRiskFloorLimit": {
            "format": "int32",
            "type": "integer",
            "description": "Valor máximo de verificação para autorização offline das transações. As transações realizadas com a leitura do Chip EMV e com valor acima do “Floor limit”, deverão ser autorizadas no modo online.\n"
          },
          "NatEmvConctactRiskMinValue": {
            "format": "int32",
            "type": "integer",
            "description": "Valor mínimo para o cálculo de seleção aleatória para autorização offline. Conforme processo definido na Especificação EMV.\n"
          },
          "NatEmvConctactRiskMinPercent": {
            "format": "int32",
            "type": "integer",
            "description": "Porcentagem mínima para seleção aleatória. Utilizar apenas o conteúdo do último byte.\n"
          },
          "NatEmvConctactRiskMaxPercent": {
            "format": "int32",
            "type": "integer",
            "description": "Porcentagem máxima para seleção aleatória. Utilizar apenas o conteúdo do último byte.\n"
          },
          "IntEmvConctactRiskFloorLimit": {
            "format": "int32",
            "type": "integer",
            "description": "Valor máximo de verificação para autorização offline das transações. As transações realizadas com a leitura do Chip EMV e com valor acima do “Floor limit”, deverão ser autorizadas no modo online.\n"
          },
          "IntEmvConctactRiskMinValue": {
            "format": "int32",
            "type": "integer",
            "description": "Valor mínimo para o cálculo de seleção aleatória para autorização offline. Conforme processo definido na Especificação EMV.\n"
          },
          "IntEmvConctactRiskMinPercent": {
            "format": "int32",
            "type": "integer",
            "description": "Porcentagem mínima para seleção aleatória. Utilizar apenas o conteúdo do último byte.\n"
          },
          "IntEmvConctactRiskMaxPercent": {
            "format": "int32",
            "type": "integer",
            "description": "Porcentagem máxima para seleção aleatória. Utilizar apenas o conteúdo do último byte.\n"
          }
        }
      },
      "InitializeResponse": {
        "type": "object",
        "properties": {
          "MerchantId": {
            "type": "string",
            "description": "Identificador da loja\n"
          },
          "TerminalId": {
            "type": "string",
            "description": "Identificador do terminal\n"
          },
          "Acquirer": {
            "$ref": "#/components/schemas/Acquirer"
          },
          "Merchant": {
            "$ref": "#/components/schemas/Merchant"
          },
          "Bins": {
            "uniqueItems": false,
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/Bin"
            }
          },
          "Products": {
            "uniqueItems": false,
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/Product"
            }
          },
          "Emv": {
            "uniqueItems": false,
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/Emv"
            }
          },
          "Parameters": {
            "uniqueItems": false,
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/Parameter"
            }
          },
          "Issuers": {
            "uniqueItems": false,
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/Issuer"
            }
          },
          "AidParameters": {
            "type": "string",
            "description": "-\n"
          },
          "PublicKeys": {
            "type": "string",
            "description": "-\n"
          }
        }
      },
      "Issuer": {
        "type": "object",
        "properties": {
          "IssuerId": {
            "format": "int32",
            "type": "integer",
            "description": "Identificador do emissor.\n"
          },
          "IssuerName": {
            "type": "string",
            "description": "Nome do emissor.\n"
          },
          "AllowFallbackWhenChipReadingFails": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite fallback se houver erro na leitura do Chip.\n"
          },
          "AllowChargingMoedeiroFromCash": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite carga de moedeiro a partir de dinheiro em espécie.\n"
          },
          "AllowPurchaseWithCompreESaque": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite venda com Compre & Saque.\n"
          },
          "AllowOfflineFunctionExceptForEMVCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite função offline, exceto cartão EMV.\n"
          },
          "AllowTypingCardNumber": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite digitação do número do cartão.\n"
          },
          "MaskCardNumberUsingLast4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Imprimir apenas os 4 últimos dígitos do cartão.\n"
          },
          "MaskCardNumberUsingFirst6AndLas4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Imprimir os 6 primeiros e os 4 últimos dígitos do cartão.\n"
          },
          "AllowPrintCardHolderBalance": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite imprimir o saldo do portador.\n"
          },
          "AllowDisplayCardHolderBalance": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite exibir no display o saldo do portador.\n"
          },
          "Option03BiAllowPrintingPartialCardNumberInReceipt07": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite impressão parcial do número do cartão no comprovante das transações.\n"
          },
          "RestrictSaleWithDuplicateValueWhenPostdated": {
            "type": "boolean",
            "readOnly": true,
            "description": "Impede venda com valor duplicado para pré-datamento.\n"
          },
          "RestrictSaleWithDuplicateValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Impede venda com valor duplicado.\n"
          },
          "RequiresPassword": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita senha.\n"
          },
          "InterpretsLastDigitOfSecurityCode": {
            "type": "boolean",
            "readOnly": true,
            "description": "Interpreta último dígito do Código de Serviço.\n"
          },
          "RequiresPasswordExceptForEMVCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita senha, exceto cartão EMV.\n"
          },
          "EnableAdditionalSecurityCodeOptions_Unreadable_NoCode": {
            "type": "boolean",
            "readOnly": true,
            "description": "Habilita opções “Ilegível” e “Não Possui” para Código de Segurança.\n"
          },
          "RequiresSecurityCodeWhenMagneticTrackIsRead": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita Código de Segurança na leitura de trilha.\n"
          },
          "RequiresSecurityCodeWhenCardNumberIsTyped": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita Código de Segurança para cartão digitado.\n"
          },
          "RequiresTypingLast4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita digitação dos últimos 4 dígitos.\n"
          },
          "AllowCaptureOfFirstInstallmentValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite captura do valor da primeira parcela.\n"
          },
          "AllowCaptureOfDownpaymentValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite captura do valor de entrada.\n"
          },
          "AllowGuaranteeHandling": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite tratamento de garantia.\n"
          },
          "AllowPostdatingTheFirstInstallmentForSaleAndCDCQuery": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite pré-datar a primeira parcela para venda e consulta CDC.\n"
          },
          "AllowPostdating": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite pré-datamento.\n"
          },
          "AllowCDCSale": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite venda CDC.\n"
          },
          "AllowFinancingByStore": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite financiamento pela Loja.\n"
          },
          "AllowFinancingByCreditCardCompany": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite financiamento pela Administradora.\n"
          },
          "RequiresChipReader": {
            "type": "boolean",
            "readOnly": true,
            "description": "Exige a existência de Leitor de Chip.\n"
          },
          "RequiresPinpad": {
            "type": "boolean",
            "readOnly": true,
            "description": "Exige a existência de PIN-pad.\n"
          },
          "LimitDayforReversal": {
            "format": "int32",
            "type": "integer",
            "description": "Data limite em dias para permitir Cancelamento.\n"
          },
          "LimitValueforReversal": {
            "type": "string",
            "description": "Valor máximo para Cancelamento.\n"
          },
          "LimitPercentforReversal": {
            "format": "int64",
            "type": "integer",
            "description": "Percentual máximo para Cancelamento.\n"
          },
          "IssuerNameForDisplay": {
            "type": "string",
            "description": "Nome do Issuer para o Display.\n"
          },
          "IssuerNameForPrint": {
            "type": "string",
            "description": "Nome do Issuer para Impressão.\n"
          }
        }
      },
      "Merchant": {
        "type": "object",
        "properties": {
          "MerchantId": {
            "type": "string",
            "description": "Código do Lojista na PayStore, definido no momento da criação do lojista.\n"
          },
          "NetworkName": {
            "type": "string",
            "description": "Nome da rede da sub-adquirente cadastrado pelo Gestor da PayStore.\n"
          },
          "MerchantName": {
            "type": "string",
            "description": "Nome fantasia do lojista, definido no momento da criação do mesmo no portal da PayStore.\n"
          },
          "MerchantAddress": {
            "type": "string",
            "description": "Endereço do lojista obtido a partir da digitação do CEP momento da criação do mesmo no portal da PayStore.\n"
          },
          "NationalId": {
            "type": "string",
            "description": "CPF ou CNPJ, definido no momento da criação do Lojista no portal da PayStore.\n"
          }
        }
      },
      "Parameter": {
        "type": "object",
        "properties": {
          "Currency": {
            "type": "string",
            "description": "-\n"
          },
          "AllowFallbackWhenChipReadingFails": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite fallback se houver erro na leitura do Chip.\n"
          },
          "AllowChargingMoedeiroFromCash": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite carga de moedeiro a partir de dinheiro em espécie.\n"
          },
          "AllowPurchaseWithCompreESaque": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite venda com Compre & Saque.\n"
          },
          "AllowOfflineFunctionExceptForEMVCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite função offline, exceto cartão EMV.\n"
          },
          "AllowTypingCardNumber": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite entrada manual do número do cartão.\n"
          },
          "MaskCardNumberUsingLast4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Imprimir apenas os 4 últimos dígitos do cartão.\n"
          },
          "MaskCardNumberUsingFirst6AndLas4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Imprimir os 6 primeiros e os 4 últimos dígitos do cartão.\n"
          },
          "AllowPrintCardHolderBalance": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite imprimir o saldo do portador.\n"
          },
          "AllowDisplayCardHolderBalance": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite exibir no display o saldo do portador.\n"
          },
          "AllowPrintingPartialCardNumberInReceipt": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite impressão parcial do número do cartão no comprovante das transações.\n"
          },
          "RestrictSaleWithDuplicateValueWhenPostdated": {
            "type": "boolean",
            "readOnly": true,
            "description": "Impede venda com valor duplicado para pré-datamento.\n"
          },
          "RestrictSaleWithDuplicateValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Impede venda com valor duplicado.\n"
          },
          "RequiresPassword": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita senha.\n"
          },
          "InterpretsLastDigitOfSecurityCode": {
            "type": "boolean",
            "readOnly": true,
            "description": "Interpreta último dígito do Código de Serviço.\n"
          },
          "RequiresPasswordExceptForEMVCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita senha, exceto cartão EMV.\n"
          },
          "EnableAdditionalSecurityCodeOptions_Unreadable_NoCode": {
            "type": "boolean",
            "readOnly": true,
            "description": "Habilita opções “Ilegível” e “Não Possui” para Código de Segurança.\n"
          },
          "RequiresSecurityCodeWhenMagneticTrackIsRead": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita Código de Segurança na leitura de trilha.\n"
          },
          "RequiresSecurityCodeWhenCardNumberIsTyped": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita Código de Segurança para cartão digitado.\n"
          },
          "RequiresTypingLast4Digits": {
            "type": "boolean",
            "readOnly": true,
            "description": "Solicita digitação dos últimos 4 dígitos.\n"
          },
          "CapturesServiceFee": {
            "type": "boolean",
            "readOnly": true,
            "description": "Captura Taxa de Serviço.\n"
          },
          "AllowCancellationWithValueGreaterThanTheValueOfTheSale": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite valor do Cancelamento maior que o valor da venda original.\n"
          },
          "CaptureBoardingFee": {
            "type": "boolean",
            "readOnly": true,
            "description": "Captura Taxa de Embarque.\n"
          }
        }
      },
      "Product": {
        "type": "object",
        "properties": {
          "ProductId": {
            "format": "int32",
            "type": "integer",
            "description": "Identificador do produto.\n"
          },
          "ProductName": {
            "type": "string",
            "description": "Nome do produto.\n"
          },
          "ProductType": {
            "format": "int32",
            "type": "integer",
            "description": "Admite os seguintes valores\n<br>0 - CREDITO\n<br>1 – DEBITO\n"
          },
          "BrandId": {
            "type": "string",
            "description": "Identificador da bandeira do cartão.\n"
          },
          "AllowTransactionWithContactlessCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite Transação com Cartão Sem Contato.\n"
          },
          "IsFinancialProduct": {
            "type": "boolean",
            "readOnly": true,
            "description": "Produto Financeiro.\n"
          },
          "AllowOfflineAuthorizationForEMVCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite Autorização Offline EMV.\n"
          },
          "AllowReprintReceipt": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite Reimpressão do comprovante.\n"
          },
          "AllowPrintReceipt": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite Impressão do comprovante.\n"
          },
          "AllowOfflineAuthorizationForContactlessCard": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite Autorização Offline para Cartão Sem Contato.\n"
          },
          "AllowCancel": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite Cancelamento.\n"
          },
          "AllowUndo": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite Desfazimento.\n"
          },
          "AllowCaptureOfFirstInstallmentValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite captura do valor da primeira parcela.\n"
          },
          "AllowCaptureOfDownpaymentValue": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite captura do valor de entrada.\n"
          },
          "AllowGuaranteeHandling": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite tratamento de garantia.\n"
          },
          "AllowPostdatingTheFirstInstallmentForSaleAndCDCQuery": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite pré-datar a primeira parcela para venda e consulta CDC.\n"
          },
          "AllowPostdating": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite pré-datamento.\n"
          },
          "AllowCDCSale": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite venda CDC.\n"
          },
          "AllowFinancingByStore": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite financiamento pela Loja.\n"
          },
          "AllowFinancingByCreditCardCompany": {
            "type": "boolean",
            "readOnly": true,
            "description": "Permite financiamento pela Administradora.\n"
          },
          "MaximumNumberOfInstallmentsWhenFinancingByCreditCardCompany": {
            "format": "int32",
            "type": "integer",
            "description": "Número máximo de parcelas para financiamento ADM.\n"
          },
          "MaximumNumberOfInstallmentsWhenFinancingByStore": {
            "format": "int32",
            "type": "integer",
            "description": "Número máximo de parcelas para financiamento Loja.\n"
          },
          "MaximumNumberOfinstallmentsForSaleAndCDCQuery": {
            "format": "int32",
            "type": "integer",
            "description": "Número máximo de parcelas para venda e consulta CDC.\n"
          },
          "MinimumNumberOfInstallmentsWhenFinancingByStore": {
            "format": "int64",
            "type": "integer",
            "description": "Valor mínimo de parcelas para financiamento Loja.\n"
          },
          "SaleGuaranteeType": {
            "type": "string",
            "description": "Tipo de Garantia para o Pré-datado.\n<br>Admite os seguintes valores\n<br>00 – Não permite tratamento de Garantia (Venda Garantida);\n<br>05 – Permite transações Pré-datadas Garantidas;\n<br>07 – Permite transações Pré-datadas Garantidas e Sem Garantia.\n"
          },
          "PostdatedDayCountLimit": {
            "format": "int32",
            "type": "integer",
            "description": "Limite máximo em dias para pré-datar a partir da data atual.\n<br>00 – Não aceita.\n<br>XX - Pré-datado.\n"
          },
          "FirstInstallmentDayCountLimit": {
            "format": "int32",
            "description": "Limite da data de primeira parcela.\n<br>00 – Não aceita.\n<br>XX - Limite de dias.\n",
            "type": "integer"
          }
        }
      }
    },
    "responses": {
      "401": {
        "description": "Erro nas credenciais\n"
      },
      "403": {
        "description": "Operação não permitida\n"
      }
    },
    "headers": {
      "ExpiresAfter": {
        "schema": {
          "type": "string",
          "format": "date-time"
        },
        "description": "date in UTC when token expires\n"
      }
    },
    "securitySchemes": {
      "oAuth2Password": {
        "type": "oauth2",
        "flows": {
          "clientCredentials": {
            "tokenUrl": "https://authsandbox.braspag.com.br/oauth2/token",
            "scopes": {
              "Administrator": "Admin everything",
              "AnalyticsApiOverview": "See the analytics",
              "AdminBackoffice": "Use the backoffice"
            }
          }
        }
      }
    }
  }
}
```
