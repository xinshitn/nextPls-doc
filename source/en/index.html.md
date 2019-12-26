---
title: API PandaRemit

language_tabs: # must be one of https://git.io/vQNgJ
  - json
  - shell
  - java

toc_footers:
  - <a href='#'>Contact Us</a>
  - <a href='https://www.pandaremit.com'>Powered by PandaRemit</a>

includes:

search: true
---

# Version 1.0.0
## Introduction

Welcome API for PandaRemit!

# Getting Started
## Authentication

Any Pandaremit Partner that will use the API will be provided with a Merchant Id. This Merchant Id will be used in all the messages sent to API and will be used in the authentication process.

A corresponding “Secret Key” will be assigned for each Merchant Id. The “Secret Key” will be used to sign each message sent to/received from the API to ensure its integrity. The signing process is detailed in `signature`.

<aside class="notice">
You must send <code>signature</code> in every request.
</aside>

## Keys

Definition of abbreviated values found in the documentation.

Parameter | Type | Description
--------- | ------- | -----------
R | string | value for this field is required
O | string | value for this field is optional
C | string | The requirement for this field is conditional based on other field values

# Remitly API
## API List of Remitly

There are SIX methods that Remitly provides, they are:

Parameter | Type | Description
--------- | ------- | -----------
1 | Create customer | Create a remittance account
2 | Update customer | Update the remittance account
3 | Get customer | Query remittance account information
4 | Create transaction | Create the remittance transaction
5 | Get payment | Query the payment method of the transaction
6 | Get transaction | Query information of the transaction

## Create customer
### HTTP Request
<span class="http-method post">POST</span> `/partner/customer/create`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "attributes": {
            "first-name": "Sam", 
            "last-name": "Samuel", 
            "contact-phone": "+19421779999", 
            "date-of-birth": "1989-09-15", 
            "last-4-ssn": "1234", 
            "address": {
                "zip-code": "20030", 
                "street": "clifford road", 
                "city": "Los Angeles", 
                "state": "California"
            }
        }
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED "
}
```
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "attributes": {
                 "first-name": "Sam", 
                 "last-name": "Samuel", 
                 "contact-phone": "+19421779999", 
                 "date-of-birth": "1989-09-15", 
                 "last-4-ssn": "1234", 
                 "address": {
                     "zip-code": "20030", 
                     "street": "clifford road", 
                     "city": "Los Angeles", 
                     "state": "California"
                 }
             }
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED "
     }'
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | String | customer's unique identification number in Pandaremit | R
/data/attributes/first-name | String | customer's firstname | R
/data/attributes/last-name | String | customer's lastname | R
/data/attributes/contact-phone | E.64 | contact number for the customer | R
/data/attributes/date-of-birth | ISO8601 | customer's date of birth | R
/data/attributes/last-4-ssn | Integer | the last 4 digits of customer's SSN | R
/data/attributes/address/zip-code | Integer | zipcode of residential address | R
/data/attributes/address/street | String | residential address | R
/data/attributes/address/city | String | city of residence | R
/data/attributes/address/state | String | state of residence | R
/signature | String | computed signature | R

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/data/reference-id | String | customer's unique identification number in Pandaremit
/data/attributes/customer-id | String | customer's unique identification number in Remitly（If status equals Refused, it can be null）
/data/attributes/status | String | Account Opening Result,Pretreatment results：<br/> Approved：the account has been opened successfully and remittances can be made.<br/> Processing：In the process of auditing<br/> Refused：Refusal to open an account<br/> Waiting replenish：additional materials are required to complete account opening.
/data/attributes/message | String | Description information of the text
/data/verification/ssn-itin/status | String | Pretreatment results：<br/> Awaiting upload<br/> Awaiting approval<br/> Approved<br/> No required
/data/verification/ssn-itin/required | Boolean | Pretreatment results：<br/> True<br/> False
/data/verification/identity/status | String | Pretreatment results：<br/> Awaiting upload<br/> Awaiting approval<br/> Approved<br/> No required
/data/verification/ identity/required | Boolean | Pretreatment results：<br/> True<br/> False
/data/verification/ address/status | String | Pretreatment results：<br/> Awaiting upload<br/> Awaiting approval<br/> Approved<br/> No required
/data/verification/ address/required | Boolean | Pretreatment results：<br/> True<br/> False
/data/verification/ funding/status | String | Pretreatment results：<br/> Awaiting upload<br/> Awaiting approval<br/> Approved<br/> No required
/data/verification/funding/required | Boolean | Pretreatment results：<br/> True<br/> False
/signature | String | Computed signature

## Update customer
### HTTP Request
<span class="http-method post">POST</span> `/partner/customer/update`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "attributes": {
            "customer-id": "92812", 
            "contact-phone": "+19421779999", 
            "ssn": "8888888", 
            "itin": "6666666", 
            "address": {
                "zip-code": "20030", 
                "street": "clifford road", 
                "city": "Los Angeles", 
                "state": "California"
            }, 
            "document": {
                "identity": {
                    "file-type": "drivers_licence", 
                    "issuing-country": "USA", 
                    "number": "GT005", 
                    "expiry": "2019-12-28", 
                    "image-base64": "QEWIJILOOIKKIPPKMJWUIEW……."
                }, 
                "address": {
                    "file-type": "bank_statement", 
                    "image-base64": "QEWIJILOOIKKIPPKMJWUIEW……."
                }, 
                "funding": {
                    "file-type": "tax_summary", 
                    "image-base64": "QEWIJILOOIKKIPPKMJWUIEW……."
                }
            }
        }
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED "
}
```
```shell
curl -X POST https://open.remitly.com/partner/customer/update
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "attributes": {
                 "customer-id": "92812", 
                 "contact-phone": "+19421779999", 
                 "ssn": "8888888", 
                 "itin": "6666666", 
                 "address": {
                     "zip-code": "20030", 
                     "street": "clifford road", 
                     "city": "Los Angeles", 
                     "state": "California"
                 }, 
                 "document": {
                     "identity": {
                         "file-type": "drivers_licence", 
                         "issuing-country": "USA", 
                         "number": "GT005", 
                         "expiry": "2019-12-28", 
                         "image-base64": "QEWIJILOOIKKIPPKMJWUIEW……."
                     }, 
                     "address": {
                         "file-type": "bank_statement", 
                         "image-base64": "QEWIJILOOIKKIPPKMJWUIEW……."
                     }, 
                     "funding": {
                         "file-type": "tax_summary", 
                         "image-base64": "QEWIJILOOIKKIPPKMJWUIEW……."
                     }
                 }
             }
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED "
     }'
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | Integer | customer's unique identification number in Pandaremit | R
/data/attributes/customer-id | Integer | customer's unique identification number in Remitly | R
/data/attributes/contact-phone | E.64 | contact number for the customer | O
/data/attributes/ssn | String | full ssn | O
/data/attributes/itin | String | full itin  | O
/data/attributes/address/zip-code | Integer | zipcode of residential address | O
/data/attributes/address/street | String | residential address | O
/data/attributes/address/city | String | city of residence | O
/data/attributes/address/state | String | state of residence | O
/data/attributes/document/identity/file-type | String | The type of document the file:<br> ID<br> Drivers_Licence<br> Passport | C
/data/attributes/document/identity/issuing-country | String | The code of country is the three letters codes (ISO alpha-3) | C
/data/attributes/document/identity/number | String | The document numbe | C
/data/attributes/document/identity/expiry | ISO8601 | Expiry date as per the document (YYYY-MM-DD) | C
/data/attributes/document/identity/image-base64 | String | The base64 image string of the document | C
/data/attributes/document/address/file-type | String | The type of document the file:<br> Bank_Statement<br> Utility_Bill | C
/data/attributes/document/address/image-base64 | String | The base64 image string of the document | C
/data/attributes/document/funding/file-type | String | The type of document the file:<br> Bank_Statement<br> Pay_Slip<br> Tax_Summary | C
/data/attributes/document/funding/image-base64 | String | The base64 image string of the document | C
/signature | String | computed signature  | R

### Response Body
Pointer | Type | Description 
--------- | ------- | ------- 
/data/reference-id | String | customer's unique identification number in Pandaremit
/data/attributes/customer-id | String | customer's unique identification number in Remitly（If status equals Refused, it can be null）
/data/attributes/status | String | Account Opening Result,Pretreatment results：<br> Approved：the account has been opened successfully and remittances can be made.<br> Processing：In the process of auditing<br> Refused：Refusal to open an account<br> Waiting replenish：additional materials are required to complete account opening.
/data/attributes/message | String | Description information of the text
/data/verification/ssn-itin/status | String | Pretreatment results：<br> Awaiting upload<br> Awaiting approval<br> Approved<br> No required
/data/verification/ssn-itin/required | Boolean | Pretreatment results：<br> True<br> False
/data/verification/identity/status | String | Pretreatment results：<br> Awaiting upload<br> Awaiting approval<br> Approved<br> No required
/data/verification/identity/required | Boolean | Pretreatment results：<br> True<br> False
/data/verification/address/status | String | Pretreatment results：<br> Awaiting upload<br> Awaiting approval<br> Approved<br> No required
/data/verification/address/required | Boolean | Pretreatment results：<br> True<br> False
/data/verification/ funding/status | String | Pretreatment results：<br> Awaiting upload<br> Awaiting approval<br> Approved<br> No required
/data/verification/funding/required | Boolean | Pretreatment results：<br> True<br> False
/signature | String | Computed signature

## Get customer
### HTTP Request
<span class="http-method post">POST</span> `/partner/customer/get`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "customer-id": "9999821"
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED "
}
```
```shell
curl -X POST https://open.remitly.com/partner/customer/get
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "customer-id": "9999821"
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED "
     }'
```

### Request Body

Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | String | customer's unique identification number in Pandaremit | R
/data/customer-id | String | customer's unique identification number in Remitly | R
/signature | String | Computed signature | R

### Response Body

Pointer | Type | Description
--------- | ------- | -------
/data/reference-id | String | customer's unique identification number in Pandaremit
/data/attributes/customer-id | String | customer's unique identification number in Remitly
/data/attributes/status | String | Account Opening Result,Pretreatment results：<br>Approved：the account has been opened successfully and remittances can be made.<br>Processing：In the process of auditing<br>Refused：Refusal to open an account<br>Waiting replenish：additional materials are required to complete account opening.
/data/attributes/message | String | Description information of the text
/data/verification/ssn-itin/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/ssn-itin/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/ identity/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/identity/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/ address/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/address/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/ funding/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/funding/required | Boolean | Pretreatment results：<br>True<br>False
/signature | String | Computed signature

## Create Transaction
### HTTP Request
<span class="http-method post">POST</span> `/partner/transaction/create`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "attributes": {
            "customer-id": "1100001", 
            "send-currency": "AUD", 
            "send-amount": "1000.00", 
            "payout-currency": "CNY", 
            "payout-amount": "4860.00", 
            "fee": "5.00", 
            "debit-amount": "1005.00", 
            "fx-rate": "4.86", 
            "reason": "Im sending money to myself"
        }, 
        "beneficiary": {
            "first-name": "Qian", 
            "last-name": "Li", 
            "account-number": "4564561436511563136"
        }
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED "
}
```
```shell
curl -X POST https://open.remitly.com/partner/transaction/create
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "attributes": {
                 "customer-id": "1100001", 
                 "send-currency": "AUD", 
                 "send-amount": "1000.00", 
                 "payout-currency": "CNY", 
                 "payout-amount": "4860.00", 
                 "fee": "5.00", 
                 "debit-amount": "1005.00", 
                 "fx-rate": "4.86", 
                 "reason": "Im sending money to myself"
             }, 
             "beneficiary": {
                 "first-name": "Qian", 
                 "last-name": "Li", 
                 "account-number": "4564561436511563136"
             }
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED "
     }'
```

### Request Body

Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | String | the serial number of the transaction in pandaremit | R
/data/attributes/customer-id | String | customer's unique identification number in Remitly | R
/data/attributes/send-currency | String | The currency of the send amount | R
/data/attributes/send-amount | decimal | The amount the customer wants to transfer | R
/data/attributes/payout-currency | String | The currency the beneficiary will receive | R
/data/attributes/payout-amount | decimal | The amount the beneficiary will receive in CNY | R
/data/attributes/fee | decimal | Fee charged to the customer for the transaction | R
/data/attributes/debit-amount | decimal | The amount the customer need to pay | R
/data/attributes/fx-rate | decimal | The rate offered to the customer | R
/data/attributes/reason | String | Purpose of the transaction | R
/data/beneficiary/first-name | String | first name of the benficiary | R
/data/beneficiary/last-name | String | last name of benficiary | R
/data/beneficiary/account-number | Integer | Beneficiary bank account number | R
/signature | String | Computed signature | R

### Response Body

Pointer | Type | Description
--------- | ------- | -------
/data/reference-id | String | the serial number of the transaction in pandaremit
/data/attributes/customer-id | String | The customer's unique identification number in Remitly
/data/attributes/transaction-id | String | The transaction's unique identification number in Remitly
/data/attributes/debit-amount | decimal | The amount the customer need to pay
/data/attributes/debit-status | String | The payment status of the transactions：<br>Awaiting funding<br>Funds received<br>Refunded
/data/attributes/status | String | The status of the transactions：<br>Invalid<br>Rejected<br>Pending<br>Holding<br>Reviewing<br>Approved<br>Cancelled
/data/attributes/message | String | Description information of the text
/data/beneficiary/account-number | Integer | Beneficiary bank account number
/data/verification/ssn-itin/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/ssn-itin/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/ identity/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/identity/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/address/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/address/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/funding/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/funding/required | Boolean | Pretreatment results：<br>True<br>False
/data/payment/online/funding-url | String | The funding URL link for online payment
/data/payment/online/effective-time | Integer | Effective deadline（UTC）
/data/payment/bank-transfer/account-name | String | The funding account name for bank_transfer only
/data/payment/bank-transfer/account-number | String | The funding account number for bank_transfer only
/data/payment/bank-transfer/reference | String  | The funding reference for bank_transfer only

## Get Payment
### HTTP Request
<span class="http-method post">POST</span> `/partner/transaction/getpayment`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "customer-id": "9999821", 
        "transaction-id": "938918013411"
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED "
}
```
```shell
curl -X POST https://open.remitly.com/partner/transaction/getpayment
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "customer-id": "9999821", 
             "transaction-id": "938918013411"
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED "
     }'
```

### Request Body

Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | String | the serial number of the transaction in pandaremit | R
/data/customer-id | String | The customer's unique identification number in Remitly | R
/data/transaction-id | String | The transaction's unique identification number in Remitly | R
/signature | String | Computed signature | R

### Response Body

Pointer | Type | Description
--------- | ------- | -------
/data/payment/online/funding-url | String | The funding URL link for online payment
/data/payment/online/effective-time | String | Effective deadline（UTC）
/data/payment/bank-transfer/account-name | String | The funding account name for bank_transfer only
/data/payment/bank-transfer/account-number | String | The funding account number for bank_transfer only
/data/payment/bank-transfer/reference | String | The funding reference for bank_transfer only

## Get transaction
### HTTP Request
<span class="http-method post">POST</span> `/partner/transaction/get`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "customer-id": "9999821", 
        "transaction-id": "938918013411"
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED "
}
```
```shell
curl -X POST https://open.remitly.com/partner/transaction/get
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "customer-id": "9999821", 
             "transaction-id": "938918013411"
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED "
     }'
```

### Request Body

Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | String | the serial number of the transaction in pandaremit | R
/data/customer-id | String | The customer's unique identification number in Remitly | R
/data/transaction-id | String | The transaction's unique identification number in Remitly | R
/signature | String | Computed signature | R

### Response Body

Pointer | Type | Description
--------- | ------- | -------
/data/reference-id | String | the serial number of the transaction in pandaremit
/data/attributes/customer-id | String | The customer's unique identification number in Remitly
/data/attributes/transaction-id | String | The transaction's unique identification number in Remitly
/data/attributes/debit-amount | decimal | The amount the customer need to pay
/data/attributes/debit-status | String | The payment status of the transactions：<br>Awaiting funding<br>Funds received<br>Refunded
/data/attributes/status | String | The status of the transactions：<br>Invalid<br>Rejected<br>Pending<br>Holding<br>Reviewing<br>Approved<br>Cancelled
/data/attributes/message | String | Description information of the text
/data/beneficiary/account-number | Integer | Beneficiary bank account number
/data/verification/ssn-itin/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/ssn-itin/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/ identity/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/identity/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/address/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/address/required | Boolean | Pretreatment results：<br>True<br>False
/data/verification/funding/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required
/data/verification/funding/required | Boolean | Pretreatment results：<br>True<br>False
/data/payment/online/funding-url | String | The funding URL link for online payment
/data/payment/online/effective-time | Integer | Effective deadline（UTC）
/data/payment/bank-transfer/account-name | String | The funding account name for bank_transfer only
/data/payment/bank-transfer/account-number | String | The funding account number for bank_transfer only
/data/payment/bank-transfer/reference | String  | The funding reference for bank_transfer only

# PandaRemit API
## API List of PandaRemit
### There are TWO methods that PandaRemit provides, they are:
Num | Api | Description
--------- | ------- | -----------
1 | Sync customer | Synchronize customer information (status)
2 | Sync transaction | Synchronize transaction status

## Sync customer
### HTTP Request
<span class="http-method post">POST</span> `/partner/customer/sync`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "customer-id": "9999821", 
        "attributes": {
            "status": "Waiting replenish"
        }, 
        "verification": {
            "ssn-itin": {
                "status": "Waiting upload", 
                "required": "true"
            }, 
            "identity": {
                "status": "Waiting upload", 
                "required": "true"
            }
        }
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED"
}
```
```shell
curl -X POST https://open.pandaremit.com/partner/customer/sync
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "customer-id": "9999821", 
             "attributes": {
                 "status": "Waiting replenish"
             }, 
             "verification": {
                 "ssn-itin": {
                     "status": "Waiting upload", 
                     "required": "true"
                 }, 
                 "identity": {
                     "status": "Waiting upload", 
                     "required": "true"
                 }
             }
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED"
     }'
```

### Request Body

Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | String | the serial number of the customer in pandaremit | R
/data/customer-id | String | The customer's unique identification number in Remitly | R
/data/attributes/status | String | Account Opening Result,Pretreatment results：<br>Approved：the account has been opened successfully and remittances can be made.<br>Processing：In the process of auditing<br>Refused：Refusal to open an account<br>Waiting replenish：additional materials are required to complete account opening. | C
/data/attributes/message | String | Description information of the text | C
/data/verification/ssn-itin/status | String | Pretreatment results：<br>Waiting upload<br>Waiting approval<br>Approved<br>No required | C
/data/verification/ssn-itin/required | String | Pretreatment results：<br>True<br>False | C
/data/verification/identity/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required | C
/data/verification/identity/required | String | Pretreatment results：<br>True<br>False | C
/data/verification/ address/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required | C
/data/verification/ address/required | String | Pretreatment results：<br>True<br>False | C
/data/verification/ funding/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required | C
/data/verification/funding/required | String | Pretreatment results：<br>True<br>False | C
/signature | String | Computed signature | R

### Response Body

Pointer | Type | Description
--------- | ------- | -------
/data/reference-id | String | the serial number of the customer in pandaremit
/data/customer-id | String | The customer's unique identification number in Remitly
/data/result/code | String | Result of sync：<br>Success<br>failure
/data/result/message | String | Description information of the text

## Sync transaction
### HTTP Request
<span class="http-method post">POST</span> `/partner/transaction/sync`

> Request Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "customer-id": "9999821", 
        "transaction-id": "9999821", 
        "attributes": {
            "debit-amount": "1000.00", 
            "debit-status": "Funds received", 
            "status": "Holding"
        }, 
        "verification": {
            "ssn-itin": {
                "status": "Waiting upload", 
                "required": "true"
            }, 
            "identity": {
                "status": "Waiting upload", 
                "required": "true"
            }
        }
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED"
}
```
```shell
curl -X POST https://open.pandaremit.com/partner/transaction/sync
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data": {
             "reference-id": "2019829921122331", 
             "customer-id": "9999821", 
             "transaction-id": "9999821", 
             "attributes": {
                 "debit-amount": "1000.00", 
                 "debit-status": "Funds received", 
                 "status": "Holding"
             }, 
             "verification": {
                 "ssn-itin": {
                     "status": "Waiting upload", 
                     "required": "true"
                 }, 
                 "identity": {
                     "status": "Waiting upload", 
                     "required": "true"
                 }
             }
         }, 
         "signature": "877CA84B66CA50234464F660B0DB51ED"
     }'
```

### Request Body

Pointer | Type | Description | Required
--------- | ------- | ------- | -----------
/data/reference-id | String | the serial number of the transaction in pandaremit | R
/data/customer-id | String | The customer's unique identification number in Remitly | R
/data/transaction-id | String | The transaction's unique identification number in Remitly | R
/data/attributes/debit-amount | decimal | The amount the customer need to pay | C
/data/attributes/debit-status | String | The payment status of the transactions：<br>Awaiting funding<br>Funds received<br>Refunded | C
/data/attributes/status | String | The status of the transactions：<br>Invalid<br>Rejected<br>Pending<br>Holding<br>Reviewing<br>Approved<br>Cancelled | O
/data/attributes/message | String | Description information of the text | R
/data/verification/ssn-itin/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required | C
/data/verification/ssn-itin/required | Boolean | Pretreatment results：<br>True<br>False | C
/data/verification/ identity/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required | C
/data/verification/identity/required | Boolean | Pretreatment results：<br>True<br>False | C
/data/verification/address/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required | C
/data/verification/address/required | Boolean | Pretreatment results：<br>True<br>False | C
/data/verification/funding/status | String | Pretreatment results：<br>Awaiting upload<br>Awaiting approval<br>Approved<br>No required | C
/data/verification/funding/required | Boolean | Pretreatment results：<br>True<br>False | C
/signature | String | Computed signature | R

### Response Body

Pointer | Type | Description
--------- | ------- | -------
/data/reference-id | String | the serial number of the transaction in pandaremit
/data/customer-id | String | The customer's unique identification number in Remitly
/data/transaction-id | String | The transaction's unique identification number in Remitly
/data/result/code | String | Result of sync：<br>Success<br>failure
/data/result/message | String | Description information of the text

# Reconciliation

Pandaremit is using CSV file as reconciliation file, sent over SFTP.
The SFTP is provided by Pandaremit, and will open a read-only account for partner. The reconciliation file will be stored on the following path: 
`/report/[year]/[month]/[day]/yyyymmdd.csv`
`Example, /report/2016/03/01/20160301.csv`

The file will be created on early T+1 day morning(00:30 AM, Beijing time), and will contains the record that be processed completely on T day.

### Columns Explanation
Column name | Explanation
--------- | ------- 
reference-id | Unique identifier of transaction, 12 digits, assigned by Partner
transaction-id | Unique identifier of transaction, 12 digits, assigned by Pandaremint
send-amount | Amount to be sent, in TranCurr currency
send-currency | Transaction currency, this field only supports RMB
cost-amount | Cost Amount of Partners
cost-currency | Cost Currency of Partners
fx-rate | Exchange Rate of Capital Conversion<br>C: present if locked exchange rate pattern <br>OR<br>C: present if unlocked exchange rate pattern and f rcv-status= RECEIVED
rcv-account | Beneficiary's UnionPay Card account number
rcv-amount | Amount of receiving remittance funds
rcv-currency | Currency of receiving remittance funds, , this field only supports RMB
rcv-status | The state of receiving remittance funds<br>RECEIVED<br>FAIL
fee | The amount of Channel cost
fee-currency | The currency of Channel cost
send-time | Instruction creation time
rcv-time | Time when the beneficiary receives the remittance


# Appendix
## Signature
The addition of signature in the messages ensures that each received message is authentic and that the content was not altered.
The signature for API is a hash based message authentication code, specifically HMAC-MD5. The signature is computed using the entire contents of the message and the Pandaremit Partner’s corresponding “Secret Key”.

The procedure to compute the signature is as follows:
1. Remove the signature field
2. Sort field records into alphabetical order based on the field/parameter name.
3. Concatenate the values of each field in the order from #2
4. Compute the signature as the HMAC-MD5 of the result of #3 and key = “Secret Key”

Example:
Secret Key - testkey

`Status request:`<br>
&nbsp;&nbsp; MerchantId – 123456780012345<br>
&nbsp;&nbsp; SearchStartDate – 201304230000<br>
&nbsp;&nbsp; SearchEndDate – 201304240000<br>
&nbsp;&nbsp; Signature – (to be computed)
  
`Step 1:` Remove the signature field<br>
&nbsp;&nbsp; MerchantId – 123456780012345<br>
&nbsp;&nbsp; SearchStartDate – 201304230000 S<br>
&nbsp;&nbsp; earchEndDate – 201304240000<br>
&nbsp;&nbsp; Signature

`Step 2:` Sort field records into alphabetical order based on the field name.<br>
&nbsp;&nbsp; MerchantId– 123456780012345<br>
&nbsp;&nbsp; earchEndDate– 201304240000<br>
&nbsp;&nbsp; SearchStartDate– 201304230000

`Step 3:` Concatenate the values of each field in the order from Step 2 123456780012345 + 201304240000 + 201304230000 = 123456780012345201304240000201304230000

`Step 4:` Compute the signature as the HMAC-MD5 of the result of Step 3 and key = testkey Signature = HMAC-MD5 of 123456780012345201304240000201304230000 and testkey Signature = 877CA84B66CA50234464F660B0DB51ED

# Errors
## HTTP Error Codes
### These are error codes that will be returned in the header of API responses

Error Code | Description
--------- | -------
400 | Bad Request -- Generic invalid request
401 | Unauthorized -- Your AuthToken is invalid or expired.
403 | Forbidden -- Requested endpoint without Authentication.
404 | Not Found -- The requested record could not be found.
405 | Method Not Allowed -- Attempting to access a method that does not exist.
422 | Unprocessable entity -- The request data is not valid.
500 | Internal Server Error -- We had a problem with our server. Try again later.
