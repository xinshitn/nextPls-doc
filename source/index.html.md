---
title: API NextPls

language_tabs:
  - json
  - shell
  - java

toc_footers:
  - <a href='#'>Contact Us</a>
  - <a href='https://www.nextpls.com'>Powered by NextPls</a>

search: true
---

# 接口版本 v1.0.0 
[For English](/en)
## 简介

欢迎选择NextPls! 我们会最大程度地保障您的资金和数据安全。

我们拥有一支优秀的开发团队。

如果想获取更多信息，可以访问我们的官方网站 [NextPls](https://www.nextpls.com) 更多地了解我们。

# 开始
## Authentication

Any Pandaremit Partner that will use the API will be provided with a Merchant Id. This Merchant Id will be used in all the messages sent to API and will be used in the authentication process.

A corresponding “Secret Key” will be assigned for each Merchant Id. The “Secret Key” will be used to sign each message sent to/received from the API to ensure its integrity. The signing process is detailed in `signature`.

<aside class="notice">
You must send <code>signature</code> in every request.
</aside>

## Keys
在下面的文档中缩写值的定义

参数 | 类型 | 描述
--------- | ------- | -----------
M | string | 必填字段
O | string | 选填字段
C | string | 有前置条件的选填或必填字段

# 基础信息
## GetCashNetworkList
获取支持现金业务的代理网络列表 
### HTTP Request
<span class="http-method post">POST</span> `/get/cashnetwork/list`

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
/data/attributes/first-name | String | customer's firstname | M
/data/attributes/last-name | String | customer's lastname | M
/data/attributes/contact-phone | E.64 | contact number for the customer | M
/data/attributes/date-of-birth | ISO8601 | customer's date of birth | M
/data/attributes/last-4-ssn | Integer | the last 4 digits of customer's SSN | M
/data/attributes/address/zip-code | Integer | zipcode of residential address | M
/data/attributes/address/street | String | residential address | M
/data/attributes/address/city | String | city of residence | M
/data/attributes/address/state | String | state of residence | M
/signature | String | computed signature | M

> Response Body:

```json
{
    "data": {
        "reference-id": "2019829921122331", 
        "attributes": {
            "customer-id": "", 
            "status": "", 
            "message": ""
        },
        "verification": {
            "ssn-itin": {
                "status": "",
                "required": ""
            },
            "identity": {
                "status": "",
                "required": ""
            },
            "address": {
                "status": "",
                "required": ""
            },
            "funding": {
                "status": "",
                "required": ""
            }
        }
    }, 
    "signature": "877CA84B66CA50234464F660B0DB51ED "
}
```

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


## GetBankNetworkList
获取支持银行业务的代理网络列表
### HTTP Request
<span class="http-method post">POST</span> `/get/bankNetwork/list`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetCountryPayMode
获取指定国家支持的支付模式 
### HTTP Request
<span class="http-method post">POST</span> `/get/countryPayMode`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

# 收汇款人信息
## DoBeneficiaryAdd
添加收款人信息
### HTTP Request
<span class="http-method post">POST</span> `/get/beneficiary/add`

> Request Body:

```json
{
    "apiName": "DO_BENEFICIARY_ADD",
    "entity": {
        "firstName": "Beneficiary_First_Name",
        "middleName": "Beneficiary_Middle_Name",
        "lastName": "Beneficiary_Last_Name",
        "telephone": "12345678910",
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
        "bankCode": "11003544",
        "bankAccountNumber": "4555556564564",
        "bankAccountName": "Beneficiary_BankName",
        "bankAddress": "Beneficiary_BankAddress"
    }
}
```

### Request Body
Pointer | Type | Description | Required
----------------------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/firstName | String | 收款人名 | M
/entity/middleName | String | 收款人中名 | O
/entity/lastName | String | 收款人姓 | M
/entity/telephone | String | 收款人手机号 | M
/entity/email | String | 收款人邮箱 | O
/entity/address1 | String | 收款人地址1 | M
/entity/address2 | String | 收款人地址2 | O
/entity/address3 | String | 收款人地址3 | O
/entity/idType | int | 收款人证件类型 | M
/entity/idNumber | String | 收款人证件号码 | M
/entity/idDesc | String | 收款人描述 | O
/entity/idIssueDate | String | 收款人证件生效时间 | O
/entity/idExpDate | String | 收款人证件失效时间 | O
/entity/birthdate | String | 收款人生日 | O
/entity/sex | String | 收款人性别 | O
/entity/nationality | String | 收款人国籍 | M
/entity/bankCode | String | 收款人账户银行编号 | M
/entity/bankAccountNumber | String | 收款人银行账户 | M
/entity/bankAccountName | String | 收款人账户银行名称 | M
/entity/bankAddress | String | 收款人账户银行地址 | O

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_ADD_R",
    "code": "200",
    "entity": {
        "firstName": "Beneficiary_First_Name",
        "middleName": "Beneficiary_Middle_Name",
        "lastName": "Beneficiary_Last_Name",
        "telephone": "12345678910",
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
        "bankCode": "11003544",
        "bankAccountNumber": "4555556564564",
        "bankAccountName": "Beneficiary_BankName",
        "bankAddress": "Beneficiary_BankAddress"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## DoBeneficiaryEdit
修改收款人信息
### HTTP Request
<span class="http-method post">POST</span> `/do/beneficiary/edit`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## DoRemitterAdd
添加汇款人信息
### HTTP Request
<span class="http-method post">POST</span> `/do/customer/add`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## DoCustomerEdit
修改汇款人信息
### HTTP Request
<span class="http-method post">POST</span> `/do/customer/edit`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## DoBeneficiaryDel
删除收款人 
### HTTP Request
<span class="http-method post">POST</span> `/do/beneficiary/del`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetBeneficiaryList
获取收款人列表
### HTTP Request
<span class="http-method post">POST</span> `/get/beneficiary/list`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetBeneficiary
获取指定收款人信息 
### HTTP Request
<span class="http-method post">POST</span> `/get/beneficiary`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetCustomer
获取汇款人信息 
### HTTP Request
<span class="http-method post">POST</span> `/get/customer`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

# 交易
## GetBalance
获取账户余额 
### HTTP Request
<span class="http-method post">POST</span> `/get/balance`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetExRate
获取汇率 
### HTTP Request
<span class="http-method post">POST</span> `/get/exRate`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## DoTransactionPre
预创建订单 
### HTTP Request
<span class="http-method post">POST</span> `/do/transaction/pre`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## DoTransactionAdd
创建订单 
### HTTP Request
<span class="http-method post">POST</span> `/do/transaction/add`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetTransactionStatus
检查交易状态
### HTTP Request
<span class="http-method post">POST</span> `/get/transaction/status`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetTransactionList
获取交易列表
### HTTP Request
<span class="http-method post">POST</span> `/get/transaction/list`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetTransaction
获取指定交易信息 
### HTTP Request
<span class="http-method post">POST</span> `/get/transaction`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
