---
title: CrossBorder API NextPls

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

NextPls CrossBorder API provide a solution for worldwide remit partners to distribution CNY in China.


# Authentication

Any NextPls Partner which use the API will be provided with a Merchant Id. This Merchant Id will be used in all messages sent to API and will be used in the authentication process. 

A corresponding "Secret Key" will be assigned for each Merchant Id. The "Secret Key" will be used to sign each message sent to/received from the API to ensure its integrity. The signing process is detailed in **[Signature](#signature)**. 

# Data Structure
All responses from PandaRemit is formatted with fixed structure.

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

'message' may not present if code=0, that means the request responded successfully. Once error occurred, code would be greater than 0, and error msg would show the error description. For more Error Code please refer **[Appendix-Errors](#errors)**.


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

# Full-Integrate API
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
area-code | string(6) | International telephone code ｜ R
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
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
Field | Type | Describe | O/M
--------- | ------- | ------- | ---------
reference-id | string(25) | Unique pipeline number of request, assigned by Partner  | R
validate-id | string(25) | Unique Number of Pre-processing created by Validate，valid for 30 minutes after generation | R
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
Field |   | Type | Description
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


# Half-Integrate API

## Validate
This method allows the partner to Register New Beneficiary.
### HTTP Request
<span class="http-method post">POST</span> `DO_BENEFICIARY_ADD`

> Request Body:

```json
{
    "apiName": "DO_BENEFICIARY_ADD",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "firstName": "Beneficiary_First_Name",
        "middleName": "Beneficiary_Middle_Name",
        "lastName": "Beneficiary_Last_Name",
        "mobile": "12345678910",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "idType": "0",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "birthdate": "01/01/1994",
        "sex": "M",
        "nationality": "HKG",
        "relationship": "3",
        "bankCode": "11003544",
        "bankAccountNumber": "4555556564564",
        "bankAccountName": "Beneficiary_BankName",
        "bankAddress": "Beneficiary_BankAddress"
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "DO_BENEFICIARY_ADD",
             "entity": {
                 "clientRemitterNo": "TEST_B001",
                 "firstName": "Beneficiary_First_Name",
                 "middleName": "Beneficiary_Middle_Name",
                 "lastName": "Beneficiary_Last_Name",
                 "mobile": "12345678910",
                 "email": "nextPls@nextPls.com",
                 "address1": "Philippines",
                 "address2": "",
                 "address3": "",
                 "idType": "0",
                 "idNumber": "PS256454165",
                 "idDesc": "",
                 "idIssueDate": "01/01/1994",
                 "idExpDate": "01/01/1994",
                 "birthdate": "01/01/1994",
                 "sex": "M",
                 "nationality": "HKG",
                 "relationship": "4",
                 "bankCode": "11003544",
                 "bankAccountNumber": "4555556564564",
                 "bankAccountName": "Beneficiary_BankName",
                 "bankAddress": "Beneficiary_BankAddress"
             }
     }'
```
```java
public class example{
    public static void main(String[] args){
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsBeneficiaryRequestDto beneficiaryRequestDto = 
                                new NextPlsBeneficiaryRequestDto();
        beneficiaryRequestDto.setClientBeneficiaryNo("TEST_B001");
        // ...
        NextPlsDoBeneficiaryAddRequest beneficiaryAddRequest = 
                NextPlsDoBeneficiaryAddRequest.build(beneficiaryRequestDto);
        client.execute(beneficiaryAddRequest);
      
    }
}
```

### Request Body
Parameter | | Type | Description | O/M
--------- | :------- | ------- | ---------- | -------
apiName | | String | DO_BENEFICIARY_ADD | M
entity | | Object | Parameter list | M
| | clientBeneficiaryNo | String(20) | Unique code for agent beneficiary | M
| | firstName | String(50) | Beneficiary First Name | M
| | middleName | String(50) | Beneficiary Middle Name | O
| | lastName | String(50) | Beneficiary Last Name | M
| | mobile | String(20) | Mobile phone Number of Beneficiary | M
| | email | String(50) | Email of Beneficiary | O
| | address1 | String(35) | Beneficiary Address1 | M
| | address2 | String(35) | Beneficiary Address2 | O
| | address3 | String(35) | Beneficiary Address3 | O
| | idType | int(2) | Type of Beneficiary Id Proof | O
| | idNumber | String(20) | Beneficiary ID Number | O
| | idDesc | String(20) | Description of Beneficiary ID,"M" only if IDType is 6 | C
| | idIssueDate | String(10) | Issue date(MM/DD/YYYY) | O
| | idExpDate | String(10) | Expiry date(MM/DD/YYYY) | O
| | birthdate | String(10) | Beneficiary BirthDate(MM/DD/YYYY) | O
| | sex | String(1) | Gender of Beneficiary | O
| | nationality | String(3) | Nationality of Beneficiary(3 Character Country ISO Code) | M
| | relationship | String(3) | Relationship with the remitter | M
| | bankCode | String(20) | Bank code for Beneficiary | C
| | bankAccountNumber | String(30) | Bank Account Number of Beneficiary | C
| | bankAccountName | String(35) | Bank Account name of Beneficiary | C
| | bankAddress | String(35) | Beneficiary Bank Address | O

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_ADD_R",
    "code": "200",
    "entity": {
       "clientBeneficiaryNo": "BE1231211",
       "beneficiaryNo": "XD201G0589941750"
    },
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_BENEFICIARY_ADD_R
code | | String | Result Code
entity | | Object | Parameter list
| | clientBeneficiaryNo | String | Unique code for partner beneficiary
| | beneficiaryNo | String | Unique code for NextPls beneficiary
msg | | String | Result message


## Remit
This method allows the partner to Edit Registered Beneficiary Profile.
### HTTP Request
<span class="http-method post">POST</span> `DO_BENEFICIARY_EDIT`

> Request Body:

```json
{
    "apiName": "DO_BENEFICIARY_EDIT",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "beneficiaryNo": "XD201G0589941750",
        "firstName": "Beneficiary_First_Name",
        "middleName": "Beneficiary_Middle_Name",
        "lastName": "Beneficiary_Last_Name",
        "mobile": "12345678910",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "idType": "0",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "birthdate": "01/01/1994",
        "sex": "M",
        "nationality": "HKG",
        "relationship": "3",
        "bankCode": "11003544",
        "bankAccountNumber": "4555556564564",
        "bankAccountName": "Beneficiary_BankName",
        "bankAddress": "Beneficiary_BankAddress"
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "DO_BENEFICIARY_ADD",
         "entity": {
             "clientRemitterNo": "TEST_B001",
             "beneficiaryNo": "XD201G0589941750",
             "firstName": "Beneficiary_First_Name",
             "middleName": "Beneficiary_Middle_Name",
             "lastName": "Beneficiary_Last_Name",
             "mobile": "12345678910",
             "email": "nextPls@nextPls.com",
             "address1": "Philippines",
             "address2": "",
             "address3": "",
             "idType": "0",
             "idNumber": "PS256454165",
             "idDesc": "",
             "idIssueDate": "01/01/1994",
             "idExpDate": "01/01/1994",
             "birthdate": "01/01/1994",
             "sex": "M",
             "nationality": "HKG",
             "relationship": "3",
             "bankCode": "11003544",
             "bankAccountNumber": "4555556564564",
             "bankAccountName": "Beneficiary_BankName",
             "bankAddress": "Beneficiary_BankAddress"
         }
     }'
```
```java
public class example{
    public static void main(String[] args){
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsBeneficiaryRequestDto beneficiaryRequestDto = new NextPlsBeneficiaryRequestDto();
        beneficiaryRequestDto.setClientBeneficiaryNo("TEST_B001");
        // ...
        NextPlsDoBeneficiaryEditRequest beneficiaryEditRequest = 
                NextPlsDoBeneficiaryEditRequest.build(beneficiaryRequestDto);
        client.execute(beneficiaryEditRequest);
      
    }
}
```

### Request Body
Parameter | | Type | Description | O/M
--------- | :------- | ------- | ---------- | -------
apiName | | String | DO_BENEFICIARY_EDIT | M
entity | | Object | Parameter list | M
| | clientBeneficiaryNo | String(20) | Unique code for agent beneficiary | M
| | beneficiaryNo | String(20) | Unique code for NextPls | M
| | firstName | String(50) | Beneficiary First Name | O
| | middleName | String(50) | Beneficiary Middle Name | O
| | lastName | String(50) | Beneficiary Last Name | O
| | mobile | String(20) | Mobile phone Number of Beneficiary | O
| | email | String(50) | Email of Beneficiary | O
| | address1 | String(35) | Beneficiary Address1 | O
| | address2 | String(35) | Beneficiary Address2 | O
| | address3 | String(35) | Beneficiary Address3 | O
| | idType | int(2) | Type of Beneficiary Id Proof | O
| | idNumber | String(20) | Beneficiary ID Number | O
| | idDesc | String(20) | Description of Beneficiary ID,"M" only if IDType is 6 | O
| | idIssueDate | String(10) | Issue date(MM/DD/YYYY) | O
| | idExpDate | String(10) | Expiry date(MM/DD/YYYY) | O
| | birthdate | String(10) | Beneficiary BirthDate(MM/DD/YYYY) | O
| | sex | String(1) | Gender of Beneficiary | O
| | nationality | String(3) | Nationality of Beneficiary(3 Character Country ISO Code) | O
| | relationship | String(3) | Relationship with the remitter | M
| | bankCode | String(20) | Bank code for Beneficiary | O
| | bankAccountNumber | String(30) | Bank Account Number of Beneficiary | O
| | bankAccountName | String(35) | Bank Account name of Beneficiary | O
| | bankAddress | String(35) | Beneficiary Bank Address | O

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_EDIT_R",
    "code": "200",
    "entity": {
       "clientBeneficiaryNo": "TEST_B001",
       "beneficiaryNo": "XD201G0589941750"
    },
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_BENEFICIARY_EDIT_R
code | | String | Result Code
entity | | Object | Parameter list
| | clientBeneficiaryNo | String | Unique code for partner beneficiary
| | beneficiaryNo | String | Unique code for NextPls beneficiary
msg | | String | Result message


# Inquiry API
## Status
This method allows the partner to Get the Balance by currency. 
### HTTP Request
<span class="http-method post">POST</span> `GET_BALANCE`

```json
{
    "apiName": "GET_BALANCE",
    "entity": {
        "currency": "HKD"
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "GET_BALANCE",
         "entity": {
             "currency": "HKD"
         }
     }'
```
```java
public class example{
    public static void main(String[] args){
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsBalanceRequestDto balanceDto = new NextPlsBalanceRequestDto();
        balanceDto.setCurrency("HKD");
        NextPlsGetBalanceRequest balanceRequest = 
                NextPlsGetBalanceRequest.build(balanceDto);
        client.execute(balanceRequest);             
      
    }
}
```

### Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_BALANCE | M
entity | | Object | Parameter list | M
| | currency | String(3) | currency | M

> Response Body:

```json
{
    "apiName": "GET_BALANCE_R",
    "code": "200",
    "entity": {
        "partnerCode": "test_client",
        "currency": "HKD",
        "balance": "1999.33"
    },
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_BALANCE_R
code | | String | Result Code
entity | | Object | Parameter list
| | partnerCode | String | Partner Code
| | currency | String | Currency
| | balance | String | Balance
msg | | String | Result message

## GetBalance
This method allows the partner to Get the last rate and lock one hour. 
### HTTP Request
<span class="http-method post">POST</span> `GET_EX_RATE`

```json
{
    "apiName": "GET_EX_RATE",
    "entity": {
        "payInCurrency": "HKD",
        "payOutCurrency": "PHP"
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "DO_TRANSACTION_PRE",
         "entity": {
             "payInCurrency": "HKD",
             "payOutCurrency": "PHP"
         }
     }'
```
```java
public class example{
    public static void main(String[] args){
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsExRateRequestDto exRateDto = new NextPlsExRateRequestDto();
        exRateDto.setPayInCurrency("HKD");
        exRateDto.setPayOutCurrency("PHP");
        NextPlsGetExRateRequest exRateRequest = NextPlsGetExRateRequest.build(exRateDto);
        client.execute(exRateRequest);
      
    }
}
```

### Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_EX_RATE | M
entity | | Object | Parameter list | M
| | payInCurrency | String(3) | Pay In Currency | M
| | payOutCurrency | String(3) | Pay Out Currency | M

> Response Body:

```json
{
    "apiName": "GET_REMITTER_R",
    "code": "200",
    "entity": {
        "payInCurrency": "HKD",
        "payOutCurrency": "PHP",
        "exRate": "7.369781"
    },
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_REMITTER_R
code | | String | Result Code
entity | | Object | Parameter list
| | payInCurrency | String | Pay In Currency
| | payOutCurrency | String | Pay Out Currency
| | exRate | String | The exchange rate
msg | | String | Result message

## Quote
This method allows the partner to preview the transfer details and keep the rate for some hours.
### HTTP Request
<span class="http-method post">POST</span> `DO_TRANSACTION_PRE`

> Request Body:

```json
{
    "apiName": "DO_TRANSACTION_PRE",
    "entity": {
        "clientTxnNo": "1000",
        "payInCurrency": "HKD",
        "payOutCurrency": "PHP",
        "transferCurrency": "HKD",
        "transferAmount": "100",
        "paymentMode": "Bank"
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "DO_TRANSACTION_PRE",
         "entity": {
             "clientTxnNo": "1000",
             "payInCurrency": "HKD",
             "payOutCurrency": "PHP",
             "transferCurrency": "HKD",
             "transferAmount": "100",
             "paymentMode": "Bank"
         }
     }'
```
```java
public class example{
    public static void main(String[] args){
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsTransactionPreRequestDto preRequestDto = new NextPlsTransactionPreRequestDto();
        preRequestDto.setClientTxnNo("1000");
        preRequestDto.setPayInCurrency("HKD");
        preRequestDto.setPayOutCurrency("PHP");
        preRequestDto.setTransferCurrency("HKD");
        preRequestDto.setTransferAmount("100");
        preRequestDto.setPaymentMode("Bank");
        NextPlsDoTransactionPreRequest preRequest = 
                NextPlsDoTransactionPreRequest.build(preRequestDto);
        client.execute(preRequest);
      
    }
}
```

### Request Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_PRE
entity | | Object | Parameter list
| | clientTxnNo | String(20) | Unique code for partner txn | M
| | payInCurrency | String(3) | Pay In Currency | M
| | payOutCurrency | String(3) | Pay Out Currency | M
| | transferCurrency | String(3) | Transfer Currency(The Fixed End, it must be payInCurrency or payOutCurrency) | M
| | transferAmount | String(18) | Transfer Amount | M
| | paymentMode | String(20) | Payment Mode | M

> Response Body:

```json
{
    "apiName": "DO_TRANSACTION_PRE_R",
    "code": "200",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000",
        "payInCurrency": "HKD",
        "payOutCurrency": "PHP",
        "payOutAmount": "736.97",
        "transferCurrency": "HKD",
        "transferAmount": "100",
        "paymentMode": "Bank",
        "exchangeRate": "7.369781",
        "commission": "0.1000",
        "totalAmount": "100.1000"
    },
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_PRE_R
code | | String | Result Code
entity | | Object | Parameter list
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | payInCurrency | String | Pay In Currency
| | payOutCurrency | String | Pay Out Currency
| | payOutAmount | String | Pay Out Amount
| | transferCurrency | String | Transfer Currency
| | transferAmount | String | Transfer Amount
| | paymentMode | String | Payment Mode
| | exchangeRate | String | Exchange Rate
| | commission | String | commission
| | totalAmount | String | Total Amount to pay
msg | | String | Result message

## AssetJournal
This method allows the partner to initiate the transfer.
### HTTP Request
<span class="http-method post">POST</span> `DO_TRANSACTION`

> Request Body:

```json
{
    "apiName": "DO_TRANSACTION",
    "entity": {
       "TxnNo": "IU201G0279816077",
       "clientTxnNo": "1000",
       "payInCountry": "HKG",
       "payOutCountry": "PHP",
       "remitterNo": "",
       "beneficiaryNo": "",
       "purposeCode": "3",
       
        "remitterFirstName": "Remitter_First_Name",
        "remitterMiddleName": "",
        "remitterLastName": "Remitter_Last_Name",
        "remitterFirstLocalName": "Remitter_First_Local_Name",
        "remitterMiddleLocalName": "",
        "remitterLastLocalName": "Remitter_Middle_Last_Name",
        "remitterMobile": "12345678910",
        "remitterSex": "M",
        "remitterBirthdate": "01/01/1994",
        "remitterEmail": "nextPls@nextPls.com",
        "remitterAddress1": "Italy",
        "remitterAddress2": "",
        "remitterAddress3": "",
        "remitterIdType": "0",
        "remitterIdNumber": "PS256454165",
        "remitterIdDesc": "",
        "remitterIdIssueDate": "01/01/1994",
        "remitterIdExpDate": "01/01/1994",
        "remitterNationality": "HKG",
        "remitterAccountNumber": "",
        "sourceIncome": "1",
        
        "beneficiaryFirstName": "Beneficiary_First_Name",
        "beneficiaryMiddleName": "",
        "beneficiaryLastName": "Beneficiary_Last_Name",
        "beneficiaryFirstLocalName": "Beneficiary_First_Local_Name",
        "beneficiaryMiddleLocalName": "",
        "beneficiaryLastLocalName": "Beneficiary_Last_Local_Name",
        "beneficiaryMobile": "12345678910",
        "beneficiarySex": "",
        "beneficiaryBirthdate": "",
        "beneficiaryEmail": "",
        "beneficiaryAddress1": "Philippines",
        "beneficiaryAddress2": "",
        "beneficiaryAddress3": "",
        "beneficiaryIdType": "0",
        "beneficiaryIdNumber": "",
        "beneficiaryIdDesc": "",
        "beneficiaryIdIssueDate": "",
        "beneficiaryIdExpDate": "",
        "beneficiaryNationality": "HKG",
        "beneficiaryRelationship": "3",
        "beneficiaryBankCode": "11003544",
        "beneficiaryBankAccountNumber": "4555556564564",
        "beneficiaryBankAccountName": "Benificiary_BankAccountName",
        "beneficiaryBankAddress": ""
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "DO_TRANSACTION",
         "entity": {
                       "TxnNo": "IU201G0279816077",
                       "clientTxnNo": "1000",
                       "payInCountry": "HKG",
                       "payOutCountry": "PHP",
                       "remitterNo": "JJ201A1131599873",
                       "beneficiaryNo": "RP122141",
                       "purposeCode": "3"
                   }
     }'
```
```java
public class example{
    public static void main(String[] args){
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsTransactionRequestDto txnRequestDto = new NextPlsTransactionRequestDto();
        txnRequestDto.setTxnNo("IU201G0279816077");
        txnRequestDto.setClientTxnNo("1000");
        txnRequestDto.setBeneficiaryNo("XD201G0589941750");
        txnRequestDto.setRemitterNo("JJ201A1131599873");
        txnRequestDto.setPurposeCode("3");
        txnRequestDto.setPayInCountry("HKG");
        txnRequestDto.setPayOutCountry("PHL");
        NextPlsDoTransactionRequest transactionAddRequest = 
                NextPlsDoTransactionRequest.build(txnRequestDto);
        client.execute(transactionAddRequest);
      
    }
}
```

### Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_TRANSACTION | M
entity | | Object | Parameter list | M
| | TxnNo | String(20) | Unique code for NextPls txn | M
| | clientTxnNo | String(20) | Unique code for partner txn | M
| | payInCountry | String(3) | Pay In Country | M
| | payOutCountry | String(3) | Pay Out Country | M
| | remitterNo | String(20) | Unique code for NextPls remitter | C
| | beneficiaryNo | String(20) | Unique code for NextPls beneficiary | C
| | purposeCode | String(2) | Purpose Code for txn | M

| | remitterFirstName | String(50) | Remitter first name | M
| | remitterMiddleName | String(50) | Remitter middle name | O
| | remitterLastName | String(50) | Remitter last name | M
| | remitterFirstLocalName | String(50) | Remitter first name | M
| | remitterMiddleLocalName | String(50) | Remitter middle name | O
| | remitterLastLocalName | String(50) | Remitter last name | M
| | remitterMobile | String(20) | Remitter mobile number | M
| | remitterEmail | String(50) | The email id of remitter | O
| | remitterAddress1 | String(35) | Remitter Address1 | M
| | remitterAddress2 | String(35) | Remitter Address2 | O
| | remitterAddress3 | String(50) | Remitter Address3 | O
| | remitterIdType | int(3) | Remitter identity type | M
| | remitterIdNumber | String(20) | ID number | M
| | remitterIdDesc | String(30) | description when ID Type=6 | C
| | remitterIdIssueDate | String(10) | ID issue date (MM/DD/YYYY) | O
| | remitterIdExpDate | String(10) | ID expiry date (MM/DD/YYYY)  | O
| | remitterBirthdate | String(10) | Remitter date of birth (MM/DD/YYYY) | O
| | remitterSex | String(1) | Remitter gender. M=Male, F=Female | O
| | remitterNationality | String(3) | Remitter Nationality(3 characters Country ISO code) | M
| | remitterAccountNumber | String(30) | Remitter account number | O
| | sourceIncome | String(2) | Remitter source of income | M

| | beneficiaryFirstName | String(50) | Beneficiary First Name | M
| | beneficiaryMiddleName | String(50) | Beneficiary Middle Name | O
| | beneficiaryLastName | String(50) | Beneficiary Last Name | M
| | beneficiaryFirstLocalName | String(50) | Beneficiary First Name | M
| | beneficiaryMiddleLocalName | String(50) | Beneficiary Middle Name | O
| | beneficiaryLastLocalName | String(50) | Beneficiary Last Name | M
| | beneficiaryMobile | String(20) | Mobile phone Number of Beneficiary | M
| | beneficiaryEmail | String(50) | Email of Beneficiary | O
| | beneficiaryAddress1 | String(35) | Beneficiary Address1 | M
| | beneficiaryAddress2 | String(35) | Beneficiary Address2 | O
| | beneficiaryAddress3 | String(35) | Beneficiary Address3 | O
| | beneficiaryIdType | int(2) | Type of Beneficiary Id Proof | O
| | beneficiaryIdNumber | String(20) | Beneficiary ID Number | O
| | beneficiaryIdDesc | String(20) | Description of Beneficiary ID,"M" only if IDType is 6 | C
| | beneficiaryIdIssueDate | String(10) | Issue date(MM/DD/YYYY) | O
| | beneficiaryIdExpDate | String(10) | Expiry date(MM/DD/YYYY) | O
| | beneficiaryBirthdate | String(10) | Beneficiary BirthDate(MM/DD/YYYY) | O
| | beneficiarySex | String(1) | Gender of Beneficiary | O
| | beneficiaryNationality | String(3) | Nationality of Beneficiary(3 Character Country ISO Code) | M
| | beneficiaryRelationship | String(3) | Relationship with the remitter | M
| | beneficiaryBankCode | String(20) | Bank code for Beneficiary | C
| | beneficiaryBankAccountNumber | String(30) | Bank Account Number of Beneficiary | C
| | beneficiaryBankAccountName | String(35) | Bank Account name of Beneficiary | C
| | beneficiaryBankAddress | String(35) | Beneficiary Bank Address | O

> Response Body:

```json
{
    "apiName": "DO_TRANSACTION_R",
    "code": "200",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000",
        "status": "TRANSACTION_ING"
    },
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_R
code | | String | Result Code
entity | | Object | Parameter list
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | status | String | The Transaction status
msg | | String | Result message

# Reconciliation

# Appendix
## Signature

# Constants

## Purpose


# Errors
### These are error codes that will be returned in the body of API responses

Error Code | Description
--------- | -------
200 | Request Success
500 | Request Fail
1006 | Without API
1007 | Invalid Signature
1010 | Missing Parameters
1011 | Wrong Parameters
10003 | Access denied
10102 | Business rate query error
20011 | Remitter add error: client remitter number duplicate
20051 | Beneficiary add error: client beneficiary number duplicate
21052 | Beneficiary query error: beneficiary info not exist
30001 | Transaction error: client txn number duplicate
30006 | Transaction error: time out
30007 | Transaction error: cancel already
43004 | Balance error: asset insufficient
