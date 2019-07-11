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
