---
title: CrossBorder API | NextPls

language_tabs: # must be one of https://git.io/vQNgJ
  - json
  - shell

toc_footers:
  - <a href='#'>Contact Us</a>
  - <a href='https://www.nextpls.com'>Powered by NextPls</a>

includes:

search: true
---

# Version 1.4.0

## Introduction

NextPls CrossBorder API provides a solution for worldwide remit partners to distribution CNY in China.


# Authentication

Any NextPls Partner which use the API will be provided with a Merchant Id. This Merchant Id will be used in all messages sent to API and will be used in the authentication process. 

A corresponding "Secret Key" will be assigned for each Merchant Id. The "Secret Key" will be used to sign each message sent to/received from the API to ensure its integrity. The signing process is detailed in **[Signature](#signature)**. 

# Data Structure
All responses from NextPls is formatted with fixed structure.

> Data structure:

```json
{
  "code": 0,
  "message":"success",
  "data": {},
  "signature":"……"
}
```

element | type
-------- | --------
code | number
message | string
data | JSONObject
signature | string

Signature only includes fields in ‘data’ object, but except ‘id-image-a’/’id-image-b’ in Create Customer. For signature algorithm please refer Appendix-Signature for details.

'message' may not present if code=0, that means the request responded successfully. Once error occurred, code would be greater than 0, and error msg would show the error description. For more Error Code please refer **[Errors](#errors)**.


# CrossBorder API Instruction
## Keys
Definition of abbreviated values found in the documentation

Abbreviation | Description
-------- | --------
R | value for this field is required
O | value for this field is optional
C | The requirement for this field is conditional based on other field values

## API List
 API | Description
-------|-------
Create Customer | Create customer in NextPls and return customer-id to partner
Create Beneficiary | Create Beneficiary in NextPls and return bankcard-id to partner
Validate | Validate remittance request and return fx-rate to partner
Remit | Remittance 
Status | Inquiry remittance status
Get Balance | Inquiry partner account balance
Quote | Inquiry realtime exchange rate
Asset Journal | Inquiry journals of partner account

# Full-Integration API
Full-integrate allows partner doing pre-registration of customer(sender) and beneficiary(receiver). Thus customer and beneficiary data are not necessary to pass to NextPls every time.
## Create Customer
This api allows partner doing customer registration in NextPls.
### HTTP Request
<span class="http-method post">POST</span> `customer`

> Request Body:

```json
{
  "data":{
          "timestamp":"1583308706625",
          "first-name":"SAN",
          "last-name":"ZHANG ",
          "id-number":"E3242339",
          "id-type":1,
          "expire-date":"2021-01-01",
          "phone-number":"18909871123",
          "area-code":"+86",
          "date-of-birth":"1989-10-12",
          "gender":"M",
          "occupation":"Engineer",
          "email":"max@pandaremit.com",
          "country":"Australia",
          "state":"NSW",
          "city":"Sydney",
          "district":"boxhill",
          "street":"Oxford St",
          "door-number":"12-Unit4-102",
          "postcode":"1023",
          "id-image-a":"QWERTYUIOPOIUYTRGHJJHBVBKL....",
          "id-image-b":"WIIUEIWIEJOIUYTJIWUUIIIUIL...."
      },
      "signature":"877CA84B66CA50234464F660B0DB51ED"
}
```

```shell
curl -X POST https://open.fft.com/register
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    {
      "data":{
              "timestamp":"1583308706625",
              "first-name":"SAN",
              "last-name":"ZHANG ",
              "id-number":"E3242339",
              "id-type":1,
              "expire-date":"2021-01-01",
              "phone-number":"18909871123",
              "area-code":"+86",
              "date-of-birth":"1989-10-12",
              "gender":"M",
              "occupation":"Engineer",
              "email":"max@pandaremit.com",
              "country":"Australia",
              "state":"NSW",
              "city":"Sydney",
              "district":"boxhill",
              "street":"Oxford St",
              "door-number":"12-Unit4-102",
              "postcode":"1023",
              "id-image-a":"QWERTYUIOPOIUYTRGHJJHBVBKL....",
              "id-image-b":"WIIUEIWIEJOIUYTJIWUUIIIUIL...."
          },
          "signature":"877CA84B66CA50234464F660B0DB51ED"
    }
```

### Request Body
Field | Type | Description | Required
--------- | ---------- | ---------| ---------
timestamp | number(13) | milliseconds of the request | R
first-name | string(32) | Sender first name | R
last-name | string(32) | Sender last name | R
id-number | string(20) | Identity number | R
id-type | number(2) | ID Type（1:Passport，2:Driver License 3:ID Card | R
expire-date | yyyy-MM-dd | ID expiration date | O
phone-number | string(20) | Phone number of customer | R
area-code | string(6) | International telephone code | R
date-of-birth | yyyy-MM-dd | Date of birth of customer | O
gender | string{1) | Gender: M male, F female | O
occupation | string(50) | Occupation | O
email | string(50) | Email address | O
country | string(20) | Customer country | O
state | string(50) | Customer state/province | O
city | string(50) | Customer city | O
district | string(50) | District of the city| O
street | string(200) | Street | O
door-number | string(50) | Door number | O
postcode | string(20) | Post code | O

> Response Body:

```json
{
    "code": 0,
    "data": {
        "customer-id": 100234,
        "first-name": "SAN",
        "last-name": "ZHANG",
        "phone-number": "18909871123",
        "area-code": "+86",
        "timestamp": 1583314988722
    },
    "signature": "47526E446D88B31220D288BB0F5A3506"
}
```

### Response Body
Field | Type | Description
--------|--------|--------
customer-id | number(16) | Unique identification number of customer in NextPls
first-name | string(32) | Sender first name
last-name | string(32) | Sender last name
phone-number | string(20) | Phone number of customer

## Create Beneficiary
This api allows partner doing beneficiary registration in NextPls.
### HTTP Request
<span class="http-method post">POST</span> `beneficiary`

> Request Body:

```json
{
  "data":{
        "timestamp": "1546054496345",
        "customer-id": "1001011",
        "bankcard": "6222600260001072444",
        "name": "李四/LISI",
        "id-number": "330424198710213219",
        "expire-date": "2025-10-10",
        "phone-number": "18909871123",
        "area-code": "+86",
        "date-of-birth": "1989-10-12",
        "occupation": "Engineer",
        "country": "中国",
        "state": "山东省",
        "city": "青岛市",
        "district": "崂山区",
        "street":"北京东路121号东山花园2-1-101"
    },
  "signature": "54C622FD5791097B6825D0D75898B015"
}
```

```shell
curl -X POST https://open.fft.com/register
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "timestamp": "1546054496345",
         "customer-id": "1001011",
         "bankcard": "6222600260001072444",
         "name": "李四/LISI",
         "id-number": "330424198710213219",
         "expire-date": "2025-10-10",
         "phone-number": "18909871123",
         "area-code": "+86",
         "date-of-birth": "1989-10-12",
         "occupation": "Engineer",
         "country": "中国",
         "state": "山东省",
         "city": "青岛市",
         "district": "崂山区",
         "street":"北京东路121号东山花园2-1-101"
         }
     }'
```

### Request Body
Field | Type | Description | Required
--------- | ------- | ------- | ---------- 
timestamp | number(13) | milliseconds of the request | R
customer-id | number(16) | Unique identification number of customer in NextPls | R
bankcard | string(20) | Bank card number | R
name | string(40) | Name of bank card holder in Chinese or Pinyin | R
id-number | string(20) | ID card number | R
expire-date | yyyy-MM-dd | ID expiration date | O
phone-number | string(20) | Phone number of beneficiary | R
date-of-birth | yyyy-MM-dd | Date of birth of beneficiary | O
occupation | string(50) | Occupation, job title | O
country | string(20) | Beneficiary country | O
state | string(50) | Beneficiary state/province | O
city | string(50) | Beneficiary city | O
district | string(50) | District of the city| O
street | string(200) | Street | O

> Response Body:

```json
{
  "data": {
    "bankcard-id": "1103410",
    "name": "LISI",
    "bank-name": "Bank of China",
    "bankcard": "6212261716002088880"
  },
  "signature": "C396777E089FED3195BAA13E94D2E116",
  "code": 0
}
```

### Response Body
Field | Type | Description
--------- | ------- | --------
bankcard-id | number | Unique identification number of beneficiary in NextPls
bankcard | string | Bank card number
name | string | Name of bank card holder
bank-name | string | Name of bank

## Validate
This api is used for remit params validation and get an exchange rate.
### HTTP Request
<span class="http-method post">POST</span> `validate`

> Request Body:

```json
{
    "data":{
        "rcv-account":"6212261202042853144",
        "bankcard-id":"1003435",
        "purpose":"Living expense",
        "reference-id":"20200304112221",
        "trans-amount":10000,
        "send-currency":"NZD"
    },
    "signature":"0EB34CBCDFF711C540DEE3D9565A7EF1"
}
```

```shell
curl -X POST http://qa.wotransfer.com/portal/api/validate
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data":{
                 "rcv-account":"6212261202042853144",
                 "bankcard-id":"1003435",
                 "purpose":"Living expense",
                 "reference-id":"20200304112221",
                 "trans-amount":10000,
                 "send-currency":"NZD"
         },
         "signature": "0EB34CBCDFF711C540DEE3D9565A7EF1 "
     }'
```

### Request Body
Field | Type | Description | Required
--------- | ------- | ------- | ---------- 
reference-id | string(25) | Unique pipeline number of request, assigned by Partner  | R
send-amount | number(14,2) | Amount to be sent in send currency(exclusive with trans-amount) | C
trans-amount | number(14,2) | Amount customer receives, CNY amount only(exclusive with send-amount) | C
send-currency | string(3) | Transaction currency, currency of the merchant asset account | R
bankcard-id | number | Unique identification number of the bankcard in NextPls | R
customer-id | number | Unique identification number of the customer in NextPls | R
purpose | string | Sender’s purpose of remittance. Predefined, see **[Purpose](#purpose)** | R

> Response Body:

```json
{
  "data": {
    "purpose": "Living expense",
    "fx-rate": "4.5811",
    "send-currency": "NZD",
    "validate-id": "b39ea797-fc05-4b5b-89dd-0bad48fd21e4",
    "rcv-account": "6212261202042853144",
    "bankcard-id": "1003435",
    "reference-id": "20200304112221"
  },
  "signature": "F8588EE3A538AC7CABB920E6E078E81B",
  "code": 0
}
```

### Response Body
Field | Type | Describe
--------- | ------- | -------
validate-id | string | This value is returned after passed validation. The validate-id is valid in 30 minutes and only supports use once.
reference-id | string | Unique identifier of transaction, assigned by Partner 
send-currency | string | Transaction currency
rcv-account | string | Beneficiary's Bank Card account number 
bankcard-id | number | Unique identification number of the bankCard in NextPls
fx-rate | number(12,4) | Exchange rate

## Remit
This api used for doing Remittance.
### HTTP Request
<span class="http-method post">POST</span> `remit`

> Request Body:

```json
{
	"data":{
        "purpose":"Oversea Shopping",
        "send-currency":"NZD",
        "validate-id":"b223f665-91ef-466c-ab7a-807c14d367d4",
        "rcv-account":"6212261716002039360",
        "bankcard-id":"1003435",
        "reference-id":"20200304112221",
        "trans-amount":"10000"
	},
	"signature":"B5FE506DBDE252BB70E23112DEBF7CFE"
}
```

```shell
curl -X POST http://qa.wotransfer.com/portal/api/remit
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data":{
                 "purpose":"Oversea Shopping",
                 "send-currency":"NZD",
                 "validate-id":"b223f665-91ef-466c-ab7a-807c14d367d4",
                 "rcv-account":"6212261716002039360",
                 "bankcard-id":"1003435",
                 "reference-id":"20200304112221",
                 "trans-amount":"10000"
         	},
            "signature":"B5FE506DBDE252BB70E23112DEBF7CFE"
     }'
```

### Request Body
Field | Type | Description | Required
--------- | ------- | ------- | ---------
reference-id | string(25) | Unique pipeline number of request, assigned by Partner  | R
validate-id | string(50) | Unique Number of Pre-processing created by Validate，valid for 30 minutes after generation | R
send-amount | number(14,2) | Amount to be sent in send currency(exclusive with trans-amount) | C
trans-amount | number(14,2) | Amount customer receives, CNY amount only(exclusive with send-amount) | C
send-currency | string(3) | Transaction currency, currency of the merchant asset account | R
bankcard-id | number | Unique identification number of the bankcard in NextPls | R
customer-id | number | Unique identification number of the customer in NextPls | R
purpose | string | Sender’s purpose of remittance. Predefined, see **[Purpose](#purpose)** | R

> Response Body:

```json
{
  "data": {
    "purpose": "Oversea Shopping",
    "fx-rate": "4.5811",
    "send-currency": "NZD",
    "validate-id": "b223f665-91ef-466c-ab7a-807c14d367d4",
    "transaction-id": "684832908401184768",
    "rcv-account": "6212261716002039360",
    "bankcard-id": "1003435",
    "reference-id": "20200304112221",
    "cost-currency": "NZD",
    "cost-amount": "2182.88"
  },
  "signature": "A4BCDE5B85F7793F1B063F9B044A07E5",
  "code": 0
}
```

### Response Body
Field | Type | Description
--------- | ------- | ------- 
validate-id | string | This value is returned after passed validation. The validate-id is valid in 30 minutes and only supports use once.
transaction-id | string | Unique identifier of transaction, generated by NextPls
reference-id | string | Unique identifier of transaction, assigned by Partner 
send-currency | string | Transaction currency
rcv-account | string | Beneficiary's Bank Card account number 
bankcard-id | number | Unique identification number of the bankCard in NextPls
fx-rate | number | Exchange rate
cost-currency | string | Currency of transaction cost
cost-amount | string | Amount of transaction cost

# Half-Integration API

## Validate
Validating remittance and get an exchange rate from NextPls. This API includes customer and beneficiary data with request.
### HTTP Request
<span class="http-method post">POST</span> `validate/v2`

> Request Body:

```json
{
    "data":{
    	"reference-id":"11223347",
    	"sender-first-name":"Sheng",
    	"sender-last-name":"Ma",
    	"sender-id-type":"1",
    	"sender-id-number":"ED1223345",
    	"expire-date":"2022-11-01",
    	"sender-phone-number":"123187723",
    	"sender-area-code":"+1",
    	"email":"12338798@qq02.com",
    	"country":"United States",
    	"country_code":"USA",
    	"receiver-name":"叶渊砾",
        "rcv-account":"6212261202042853144",
        "receiver-id-number":"331003199602160050",
        "receiver-phone-number":"17764525250",
        "purpose":"Support for family",
        "trans-amount":10000.00,
        "send-currency":"USD"
    },
    "signature":"811075DB6D5CDF6F4BBC218119EFB008"
}
```

```shell
curl -X POST http://qa.wotransfer.com/portal/api/validate/v2
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data":{
         	"reference-id":"11223347",
         	"sender-first-name":"Sheng",
         	"sender-last-name":"Ma",
         	"sender-id-type":"1",
         	"sender-id-number":"ED1223345",
         	"expire-date":"2022-11-01",
         	"sender-phone-number":"123187723",
         	"sender-area-code":"+1",
         	"email":"12338798@qq02.com",
         	"country":"United States",
         	"country_code":"USA",
         	"receiver-name":"叶渊砾",
             "rcv-account":"6212261202042853144",
             "receiver-id-number":"331003199602160050",
             "receiver-phone-number":"17764525250",
             "purpose":"Support for family",
             "trans-amount":10000.00,
             "send-currency":"USD"
         },
         "signature":"811075DB6D5CDF6F4BBC218119EFB008"
     }'
```

### Request Body
Field | Type | Description | Required
--------- | ------- | ------- | ---------- 
reference-id | string(25) | Unique pipeline number of request, assigned by Partner  | R
sender-first-name | string(32) | Sender first name | R
sender-last-name | string(32) | Sender last name | R
sender-id-number | string(20) | Sender identity number | R
sender-id-type | number(2) | Sender ID Type（1:Passport，2:Driver License 3:ID Card 4:Others | R
expire-date | yyyy-MM-dd | ID expiration date | O
sender-phone-number | string(20) | Phone number of customer | R
sender-area-code | string(6) | International telephone code | R
date-of-birth | yyyy-MM-dd | Date of birth of customer | O
gender | string{1) | Gender: M male, F female | O
occupation | string(50) | Occupation | O
email | string(50) | Email address | O
country | string(20) | Customer country | O
state | string(50) | Customer state/province | O
city | string(50) | Customer city | O
district | string(50) | District of the city| O
street | string(200) | Street | O
country-code | string(3) | Country code of country , e.g. USA, AUS | R
receiver-name | string(32) | Receiver full name | R
rcv-account | string(20) | Receiver bank account | R
receiver-id-number | string(20) | Receiver ID card no. | R
receiver-phone-number | string(20) | Receiver phone number | R
send-amount | number(14,2) | Amount to be sent in send currency(exclusive with trans-amount) | C
trans-amount | number(14,2) | Amount customer receives, CNY amount only(exclusive with send-amount) | C
send-currency | string(3) | Transaction currency, currency of the merchant asset account | R
purpose | string | Purpose of remittance. Predefined, see **[Purpose](#purpose)** | R

> Response Body:

```json
{
  "data": {
    "purpose": "Support for family",
    "receiver-name": "叶渊砾",
    "fx-rate": "7.0531",
    "send-currency": "USD",
    "sender-last-name": "Ma",
    "validate-id": "aecaabd7-22a0-4d8f-a636-7a1794549d4c",
    "sender-first-name": "Sheng",
    "rcv-account": "6212261202042853144",
    "reference-id": "11223347"
  },
  "signature": "B7B8054D9E9DFD2C0E87E17345435B68",
  "code": 0
}
```

### Response Body
Field | Type | Describe
--------- | ------- | --------
validate-id | string | This value is returned after passed validation. The validate-id is valid in 30 minutes and only supports use once.
reference-id | string | Unique identifier of transaction, assigned by Partner 
send-currency | string | Transaction currency
rcv-account | string | Beneficiary's Bank Card account number 
sender-last-name | string | Sender last name
sender-first-name | string | Sender first name
receiver-name | string | Receiver's full name
fx-rate | number(12,4) | Exchange rate
purpose | string | Purpose of remittance


## Remit
This api is for doing remittance. The API includes customer and beneficiary data with request.
### HTTP Request
<span class="http-method post">POST</span> `remit/v2`

> Request Body:

```json
{
    "data":{
        	"reference-id":"11223345",
        	"validate-id":"fd938d40-d02e-4f75-9f08-671cb37c3006",
        	"sender-first-name":"Sheng",
        	"sender-last-name":"Ma",
        	"sender-id-type":"1",
        	"sender-id-number":"ED1223345",
        	"expire-date":"2022-11-01",
        	"sender-phone-number":"123187723",
        	"sender-area-code":"+1",
        	"email":"12338798@qq02.com",
        	"country":"United States",
        	"country_code":"USA",
        	"receiver-name":"叶渊砾",
            "rcv-account":"6212261202042853144",
            "receiver-id-number":"331003199602160050",
            "receiver-phone-number":"17764525250",
            "purpose":"Support for family",
            "trans-amount":10000.0,
            "send-currency":"USD"
        },
        "signature":"BE77A8AA1B2FC469149EC46B9089DFEB"
}
```

```shell
curl -X POST http://qa.wotransfer.com/portal/api/remit/v2
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data":{
            "reference-id":"11223345",
            "validate-id":"fd938d40-d02e-4f75-9f08-671cb37c3006",
            "sender-first-name":"Sheng",
            "sender-last-name":"Ma",
            "sender-id-type":"1",
            "sender-id-number":"ED1223345",
            "expire-date":"2022-11-01",
            "sender-phone-number":"123187723",
            "sender-area-code":"+1",
            "email":"12338798@qq02.com",
            "country":"United States",
            "country_code":"USA",
            "receiver-name":"叶渊砾",
             "rcv-account":"6212261202042853144",
             "receiver-id-number":"331003199602160050",
             "receiver-phone-number":"17764525250",
             "purpose":"Support for family",
             "trans-amount":10000.0,
             "send-currency":"USD"
         },
         "signature":"BE77A8AA1B2FC469149EC46B9089DFEB"
     }'
```

### Request Body
Parameter | Type | Description | Required
--------- | ------- | ------- | ----------
reference-id | string(25) | Unique pipeline number of request, assigned by Partner  | R
validate-id | string(50) | This value is returned after passed validation. The validate-id is valid in 30 minutes and only supports use once. | R
sender-first-name | string(32) | Sender first name | R
sender-last-name | string(32) | Sender last name | R
sender-id-number | string(20) | Sender identity number | R
sender-id-type | number(2) | Sender ID Type（1:Passport，2:Driver License 3:ID Card 4:Others | R
expire-date | yyyy-MM-dd | ID expiration date | O
sender-phone-number | string(20) | Phone number of customer | R
sender-area-code | string(6) | International telephone code | R
date-of-birth | yyyy-MM-dd | Date of birth of customer | O
gender | string{1) | Gender: M male, F female | O
occupation | string(50) | Occupation | O
email | string(50) | Email address | O
country | string(20) | Customer country | O
state | string(50) | Customer state/province | O
city | string(50) | Customer city | O
district | string(50) | District of the city| O
street | string(200) | Street | O
country-code | string(3) | Country code of country , e.g. USA, AUS | R
receiver-name | string(32) | Receiver full name | R
rcv-account | string(20) | Receiver bank account | R
receiver-id-number | string(20) | Receiver ID card no. | R
receiver-phone-number | string(20) | Receiver phone number | R
send-amount | number(14,2) | Amount to be sent in send currency(exclusive with trans-amount) | C
trans-amount | number(14,2) | Amount customer receives, CNY amount only(exclusive with send-amount) | C
send-currency | string(3) | Transaction currency, currency of the merchant asset account | R
purpose | string | Purpose of remittance. Predefined, see **[Purpose](#purpose)** | R

> Response Body:

```json
{
    "data": {
        "purpose": "Support for family",
        "receiver-name": "叶渊砾",
        "fx-rate": "7.0531",
        "send-currency": "USD",
        "sender-last-name": "Ma",
        "validate-id": "e42d91f4-b0a6-46a5-9481-e7da0d226cc5",
        "sender-first-name": "Sheng",
        "transaction-id": "685146609809362944",
        "rcv-account": "6212261202042853144",
        "reference-id": "11223345",
        "cost-currency": "USD",
        "cost-amount": "1417.82"
      },
      "signature": "818CBD838FA4A1CD60A83003D3344568",
      "code": 0
}
```

### Response Body
Field | Type | Describe
--------- | ------- | --------
validate-id | string | This value is returned after passed validation. The validate-id is valid in 30 minutes and only supports use once.
transaction-id | string | Unique identifier of transaction, generated by NextPls
reference-id | string | Unique identifier of transaction, assigned by Partner 
send-currency | string | Transaction currency
rcv-account | string | Beneficiary's Bank Card account number 
sender-last-name | string | Sender last name
sender-first-name | string | Sender first name
receiver-name | string | Receiver's full name
fx-rate | number(12,4) | Exchange rate
purpose | string | Purpose of remittance
cost-currency | string(3) | The currency of money deducted from partner's account
cost-amount | number(14,2)) | Money cost in cost-currency

# Inquiry API
## Status
This method allows the partner to get transaction status from NextPls by . 
### HTTP Request
<span class="http-method post">POST</span> `status`

> Request body:

```json
{
    "data":{
    	"transaction-id":"685146609809362944"
    },
    "signature":"952FA0D6C0AC79F2E2AFDAC4531DBFBE"
}
```

```shell
curl -X POST http://qa.wotransfer.com/portal/api/remit/v2
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
        "data":{
            "transaction-id":"685146609809362944"
        },
        "signature":"952FA0D6C0AC79F2E2AFDAC4531DBFBE"
     }'
```

### Request Body
Field | Type | Describe | O/M
--------- | ------- | ------- | ----------
transaction-id | string | Transaction id of NextPls returned by *Remit* API.

> Response Body:

```json
{
  "data": {
    "purpose": "Support for family",
    "fee": "0.00",
    "receiver-name": "叶渊砾",
    "create-time": "2020-03-05 15:28:01",
    "fx-rate": "7.0531",
    "send-currency": "USD",
    "sender-last-name": "Ma",
    "sender-first-name": "Sheng",
    "transaction-id": "685146609809362944",
    "rcv-account": "6212***********3144",
    "reference-id": "11223345",
    "trans-amount": "10000.00",
    "fee-currency": "USD",
    "cost-currency": "USD",
    "send-amount": "1417.82",
    "cost-amount": "1417.82",
    "status": "TRANSACTION_ING"
  },
  "signature": "8E99EB087D3ADE23BBC6A793F5BA8E3D",
  "code": 0
}
```

### Response Body
Field | Type | Describe
--------- | ------- | --------
validate-id | string | This value is returned after passed validation. The validate-id is valid in 30 minutes and only supports use once.
transaction-id | string | Unique identifier of transaction, generated by NextPls
reference-id | string | Unique identifier of transaction, assigned by Partner 
send-currency | string | Transaction currency
send-amount | decimal | The amount in send currency
trans-amount | decimal | The amount that receiver receives in CNY
rcv-account | string | Beneficiary's Bank Card account number 
sender-last-name | string | Sender last name
sender-first-name | string | Sender first name
receiver-name | string | Receiver's full name
fx-rate | decimal | Exchange rate
purpose | string | Purpose of remittance
cost-currency | string | The currency of money deducted from partner's account
cost-amount | decimal | Money cost in cost-currency
fee | decimal | Fee amount of the transaction
fee-currency | string | Currency of fee
status | string | Status of transaction
create-time | string | The creation time of the txn in format *yyyy-MM-dd HH:mm:ss*
remit-time | string | Time of remittance finished in format *yyyy-MM-dd HH:mm:ss*

## GetBalance
This method allows partners to Get Balance of their account in NextPls. 

This API has no mandatory parameter in request actually, but **merchant-id** in Http Header is required.
### HTTP Request
<span class="http-method post">POST</span> `getbalance`

```json
{
	"data":{
        "timestamp":1574116548000
    },
	"signature":"D5F0E729C11E508CDD3646DA2A7E1EF9"
}
```
```shell
curl -X POST http://qa.wotransfer.com/portal/api/getbalance
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data":{
             "timestamp":1574116548000
         },
         "signature":"D5F0E729C11E508CDD3646DA2A7E1EF9"
     }'
```

### Request Body
Field | Type | Description | Required
--------- | ------- | ------- | -------
timestamp | number(13) | Milliseconds of the request, only used for signature | R

> Response Body:

```json
{
  "data": {
    "assets": [
      {
        "amount": "67.5900",
        "amount-hold": "30.2300",
        "currency": "NZD",
        "total-amount": "100.0000",
        "timestamp": 1581996524000
      },
      {
        "amount": "0.0000",
        "amount-hold": "0.0000",
        "currency": "USD",
        "total-amount": "0.0000",
        "timestamp": 1575616600000
      }
    ]
  },
  "signature": "C31ADC820A944219582EB9B08F50DC5E",
  "code": 0
}
```

### Response Body
Field |   | Type | Description
--------- | ------- | ------- |--------
assets | | Array | Asset list in different currency
| | amount | decimal | Available amount of the account in corresponding currency
| | amount-hold | decimal | Hold amount of the account in corresponding currency
| | total-amount | decimal | Total amount of the account in corresponding currency
| | currency | string | Account currency
| | timestamp | number | Milliseconds of the response

## Quote
This method allows the partner to get latest exchange rate in specific currency.
### HTTP Request
<span class="http-method post">POST</span> `quote`

> Request Body:

```json
{
    "data":{
        "sourceCurrency":"NZD"
    },
    "signature":"532ACF0372BD9DECE481BEFF4E5BCBBD"
}
```

```shell
curl -X POST http://qa.wotransfer.com/portal/api/quote
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
         "data":{
             "source-currency":"NZD"
         },
         "signature":"532ACF0372BD9DECE481BEFF4E5BCBBD"
     }'
```

### Request Body
Field | Type | Description
--------- | ------- | --------
source-currency | string(3) | The currency of fxRate that queried

> Response Body:

```json
{
  "data": {
    "update-time": 1583398474176,
    "rate": "4.3663",
    "source-currency": "NZD",
    "usd-rate": "6.9195"
  },
  "signature": "19A56F3BAFF884C387CF93EA2D3DAB4C",
  "code": 0
}
```

### Response Body
Field | Type | Description
--------- | ------- | -------
rate | decimal(12,4) | Latest exchange rate
source-currency | string | currency
usd-rate | decimal(12,4) | Base USD to CNY exchange rate
update-time | number | Exchange rate last updated time in milliseconds

## AssetJournal
This method allows the partners to retrieve the asset journals of their account with pagination.
### HTTP Request
<span class="http-method post">POST</span> `asset/journal`

> Request Body:

```json
{
	"data":{
		"start-date":"2020-03-04",
		"end-date":"2020-03-05"
	},
	"signature":"12C6E2B588347338DFB58A534CDA4E0B"
}
```
```shell
curl -X POST http://qa.wotransfer.com/portal/api/asset/journal
    -H "merchant-id:10192012192"
    -H "Content-Type: application/vnd.api+json"
    -d
    '{
     	"data":{
     		"start-date":"2020-03-04",
     		"end-date":"2020-03-05"
     	},
     	"signature":"12C6E2B588347338DFB58A534CDA4E0B"
     }'
```

### Request Body
Field | Type | Description | Required
--------- | ------- | ------- | ---------
start-date | string | Query journal start date in yyyy-MM-dd format | O
end-date | string | Query journal end date in yyyy-MM-dd format | O
page-num | number | Current inquiry page number, default 1 | O
page-size | number | The size of each page , default 20 | O

> Response Body:

```json
{
  "code": 0,
  "data": {
    "page-num": 1,
    "end-date": "2020-03-05",
    "page-size": 20,
    "total-size": 5,
    "start-date": "2020-03-04",
    "total-page": 1,
    "list": [
      {
        "update-time": "2020-03-04 10:22:40",
        "new-avail-amount": 99640.7200,
        "origin-hold-amount": 179.6400,
        "merchant-id": "11000005",
        "origin-avail-amount": 99820.3600,
        "new-hold-amount": 359.2800,
        "operation": "CREATE",
        "total-amount": "100000.0000",
        "trade-amount": 179.6400
      },
      {
        "update-time": "2020-03-04 10:29:34",
        "new-avail-amount": 92439.5000,
        "origin-hold-amount": 359.2800,
        "merchant-id": "11000005",
        "origin-avail-amount": 99640.7200,
        "new-hold-amount": 7560.5000,
        "operation": "CREATE",
        "total-amount": "100000.0000",
        "trade-amount": 7201.2200
      }]
  }
}
```

### Response Body
Field |   | Type | Description
--------- | ------- | ------- |--------
start-date |  | string | Query journal start date in yyyy-MM-dd format
end-date |  | string | Query journal end date in yyyy-MM-dd format
page-num |  | number | Current inquiry page number, default 1
page-size |  | number | The size of each page , default 20
total-page |  | number | Total pages of results
total-size |  | number | Total number of results
list | | Array | Parameter list
| | update-time | string | Asset update time
| | new-avail-amount | decimal | Available amount after operation
| | new-hold-amount | decimal | Hold amount after updating
| | origin-avail-amount | decimal | Available amount before updating
| | origin-hold-amount | decimal | Hold amount before updating
| | operation | string | The action which caused asset updating
| | total-amount | decimal | Total amount, equivalent avail-amount plus hold amount
| | trade-amount | decimal | The amount of transaction
| | merchant-id | string | The partner unique merchant id

# Reconciliation
Coming soon...

# Appendix
## Signature
The addition of signature in the messages ensures that each received message is authentic and that the content was not altered. 

The signature for API is a hash based message authentication code, specifically HMAC-MD5. The signature is computed using the entire contents of the message and the NextPls Partner’s corresponding “Secret Key”. 

The procedure to compute the signature is as follows: 

1. Remove the signature field
2. Sort field records into alphabetical order based on the field/parameter name. 
3. Concatenate the values of each field in the order from #2
4. Compute the signature as the HMAC-MD5 of the result of #3 and key = “Secret Key” 

Example:
Secret Key - testkey 

Status request: <br/>
MerchantId – 123456780012345<br/>
SearchStartDate – 202003030000<br/>
SearchEndDate – 202003060000<br/>
Signature – (to be computed) <br/>

Step 1: Remove the signature field <br/>
MerchantId – 123456780012345 <br/>
SearchStartDate – 202003030000<br/>
SearchEndDate – 202003060000<br/>
Signature 

Step 2: Sort field records into alphabetical order based on the field name.<br/>
MerchantId – 123456780012345<br/>
SearchEndDate – 202003060000<br/>
SearchStartDate – 202003030000

Step 3: Concatenate the values of each field in the order from Step 2 123456780012345 + 202003060000 + 202003030000 = 123456780012345202003060000202003030000

Step 4: Compute the signature as the HMAC-MD5 of the result of Step 3 and key = testkey Signature = HMAC-MD5 of 123456780012345202003060000202003030000 and testkey, signature is e1afc0223c539b24b2ff6947702ebe0f, 
after transfer upper case then Signature = E1AFC0223C539B24B2FF6947702EBE0F.

## Constants
### Purpose
Value | Description
-------- | --------
Salary | Customer's salary
Family Support | Support for family
Consumer goods | Consumer goods, merchandise and retail consumptions
Donations | Donations/Gifts

# Errors
These error codes would be returned in the body of API responses.

Error Code | Description
--------- | -------
199 | Signature does not match.
500 | Request fail
5013 | Missing Parameters
5014 | Wrong Parameters
10005 | Registration Error.
10109 | Rate query failed.
20014 | Sender creation failed.
21007 | The bank account already exist.
21008 | Adding receiver bank account failed.
43001 | Validation error.
43002 | Remittance hasn't been validated，or has been processed.
43003 | Remittance error.
43004 | Insufficient funds of partner account.
43005 | Inquiry account balance failed.
43006 | Inquiry transaction status failed.
43007 | Inquiry asset journals failed.
