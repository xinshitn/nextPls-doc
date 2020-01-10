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

欢迎选择 NextPls! 我们会最大程度地保障您的资金和数据安全。

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


> Response Body:

```json
{
    
}
```

<table>
	<tr>
	    <th>属性</th>
	    <th>属性值</th>
	    <th>描述</th>  
	</tr >
	<tr >
	    <td rowspan="9">type</td>
	    <td>text</td>
	    <td>单行文本输入框</td>
	</tr>
	<tr>
	    <td>password</td>
	    <td>密码输入框</td>
	</tr>
	<tr>
	    <td>radio</td>
	    <td>单选按钮</td>
	</tr>
	<tr>
	    <td>CheckBox</td>
	    <td>复选按钮</td>
	</tr>
	<tr><td>button</td>
	    <td>普通按钮</td>
	</tr>
	<tr>
	    <td>submit</td>
	    <td>提交按钮</td>
	</tr>
	<tr>
	    <td>reset</td>
	    <td>重置按钮</td>
	</tr>
	<tr>
	    <td>image</td>
	    <td>图像形式的提交按钮</td>
	</tr>
	<tr>
	    <td >file</td>
	    <td>文件域</td>
	</tr>
	<tr>
	    <td >name</td>
	    <td>用户自定义</td>
	    <td>控件名称</td>
	</tr>
	<tr>
	    <td >value</td>
	    <td >用户自定义</td>
	    <td >默认文本值</td>
	</tr>
	<tr>
	    <td >size</td>
	    <td >正整数</td>
	    <td >控件在页面中的显示宽度</td>
	</tr>
	<tr>
	    <td >checked</td>
	    <td >checked</td>
	    <td >定义选择控件默认被选中项</td>
	</tr>
	<tr>
	    <td >maxlength</td>
	    <td >正整数</td>
	    <td >控件允许输入的最多字符</td>
	</tr>
</table>

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

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
        "clientRemitterNo": "BE1231211",
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
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
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
       "clientBeneficiaryNo": "BE1231211",
       "beneficiaryNo": "BP123123"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号
/entity/beneficiaryNo | String | NextPls收款人唯一编号
/msg | String | 返回消息


## DoBeneficiaryEdit
修改收款人信息
### HTTP Request
<span class="http-method post">POST</span> `/do/beneficiary/edit`

> Request Body:

```json
{
    "apiName": "DO_BENEFICIARY_ADD",
    "entity": {
        "clientRemitterNo": "BE1231211",
        "beneficiaryNo": "BP123123",
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
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
/entity/beneficiaryNo | String | NextPls收款人唯一编号 | M
/entity/firstName | String | 收款人名 | O
/entity/middleName | String | 收款人中名 | O
/entity/lastName | String | 收款人姓 | O
/entity/telephone | String | 收款人手机号 | O
/entity/email | String | 收款人邮箱 | O
/entity/address1 | String | 收款人地址1 | O
/entity/address2 | String | 收款人地址2 | O
/entity/address3 | String | 收款人地址3 | O
/entity/idType | int | 收款人证件类型 | O
/entity/idNumber | String | 收款人证件号码 | O
/entity/idDesc | String | 收款人描述 | O
/entity/idIssueDate | String | 收款人证件生效时间 | O
/entity/idExpDate | String | 收款人证件失效时间 | O
/entity/birthdate | String | 收款人生日 | O
/entity/sex | String | 收款人性别 | O
/entity/nationality | String | 收款人国籍 | O
/entity/bankCode | String | 收款人账户银行编号 | O
/entity/bankAccountNumber | String | 收款人银行账户 | O
/entity/bankAccountName | String | 收款人账户银行名称 | O
/entity/bankAddress | String | 收款人账户银行地址 | O

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_ADD_R",
    "code": "200",
    "entity": {
       "clientBeneficiaryNo": "BE1231211",
       "beneficiaryNo": "BP123123"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------

/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号
/entity/beneficiaryNo | String | NextPls收款人唯一编号
/msg | String | 返回消息

## DoRemitterAdd
添加汇款人信息
### HTTP Request
<span class="http-method post">POST</span> `/do/remitter/add`

> Request Body:

```json
{
    "apiName": "DO_REMITTER_ADD",
    "entity": {
        "clientRemitterNo": "RE1233112",
        "firstName": "Remitter_First_Name",
        "middleName": "Remitter_Middle_Name",
        "lastName": "Remitter_Last_Name",
        "telephone": "12345678910",
        "sex": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Italy",
        "address2": "",
        "address3": "",
        "idType": "0",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "accountNumber": "",
        "sourceIncome": "1"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientRemitterNo | String | 客户端汇款人唯一编号 | M
/entity/firstName | String | 汇款人名 | M
/entity/middleName | String | 汇款人中名 | O
/entity/lastName | String | 汇款人姓 | M
/entity/telephone | String | 汇款人手机号 | M
/entity/email | String | 汇款人邮箱 | O
/entity/address1 | String | 汇款人地址1 | M
/entity/address2 | String | 汇款人地址2 | O
/entity/address3 | String | 汇款人地址3 | O
/entity/idType | int | 汇款人证件类型 | M
/entity/idNumber | String | 汇款人证件号码 | M
/entity/idDesc | String | 汇款人描述 | O
/entity/idIssueDate | String | 汇款人证件生效时间 | O
/entity/idExpDate | String | 汇款人证件失效时间 | O
/entity/birthdate | String | 汇款人生日 | O
/entity/sex | String | 汇款人性别 | O
/entity/nationality | String | 汇款人国籍 | M
/entity/accountNumber | String | 汇款人银行账号 | O
/entity/sourceIncome | String | 汇款人收入来源 | M

> Response Body:

```json
{
    "apiName": "DO_REMITTER_ADD",
    "code": "200",
    "entity": {
        "clientRemitterNo": "RE1233112",
        "remitterNo": "RP122141"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientRemitterNo | String | 客户端汇款人唯一编号
/entity/remitterNo | String | NextPls汇款人唯一编号
/msg | String | 返回消息

## DoRemitterEdit
修改汇款人信息
### HTTP Request
<span class="http-method post">POST</span> `/do/remitter/edit`

> Request Body:

```json
{
    "apiName": "DO_REMITTER_EDIT",
    "entity": {
        "clientRemitterNo": "RE1233112",
        "remitterNo": "RP122141",
        "firstName": "Remitter_First_Name",
        "middleName": "Remitter_Middle_Name",
        "lastName": "Remitter_Last_Name",
        "telephone": "12345678910",
        "sex": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "idType": "0",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "accountNumber": "",
        "sourceIncome": "1"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientRemitterNo | String | 客户端汇款人唯一编号 | M
/entity/remitterNo | String | NextPls汇款人唯一编号 | M
/entity/firstName | String | 汇款人名 | O
/entity/middleName | String | 汇款人中名 | O
/entity/lastName | String | 汇款人姓 | O
/entity/telephone | String | 汇款人手机号 | O
/entity/email | String | 汇款人邮箱 | O
/entity/address1 | String | 汇款人地址1 | O
/entity/address2 | String | 汇款人地址2 | O
/entity/address3 | String | 汇款人地址3 | O
/entity/idType | int | 汇款人证件类型 | O
/entity/idNumber | String | 汇款人证件号码 | O
/entity/idDesc | String | 汇款人描述 | O
/entity/idIssueDate | String | 汇款人证件生效时间 | O
/entity/idExpDate | String | 汇款人证件失效时间 | O
/entity/birthdate | String | 汇款人生日 | O
/entity/sex | String | 汇款人性别 | O
/entity/nationality | String | 汇款人国籍 | O
/entity/accountNumber | String | 汇款人银行账号 | O
/entity/sourceIncome | String | 汇款人收入来源 | O

> Response Body:

```json
{
    "apiName": "DO_REMITTER_EDIT",
    "code": "200",
    "entity": {
        "clientRemitterNo": "RE1233112",
        "remitterNo": "RP122141"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientRemitterNo | String | 客户端汇款人唯一编号
/entity/remitterNo | String | NextPls汇款人唯一编号
/msg | String | 返回消息


## DoBeneficiaryDel
删除收款人 
### HTTP Request
<span class="http-method post">POST</span> `/do/beneficiary/del`

> Request Body:

```json
{
    "apiName": "DO_BENEFICIARY_DEL",
    "entity": {
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "RP122141"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
/entity/BeneficiaryNo | String | NextPls收款人唯一编号 | M

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_DEL_R",
    "code": "200",
    "entity": {
        "clientBeneficiaryNo": "RE1233112",
        "beneficiaryNo": "RP122141"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号
/entity/beneficiaryNo | String | NextPls收款人唯一编号
/msg | String | 返回消息

## GetBeneficiaryList
获取收款人列表
### HTTP Request
<span class="http-method post">POST</span> `/get/beneficiary/list`

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M


### Response Body
Pointer | Type | Description
--------- | ------- | -----------

## GetBeneficiary
获取指定收款人信息 
### HTTP Request
<span class="http-method post">POST</span> `/get/beneficiary`

> Request Body:

```json
{
    "apiName": "GET_BENEFICIARY",
    "entity": {
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "RP122141"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
/entity/BeneficiaryNo | String | NextPls收款人唯一编号 | M

> Response Body:

```json
{
    "apiName": "GET_BENEFICIARY_R",
    "code": "200",
    "entity": {
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "RP122141",
        "firstName": "Beneficiary_First_Name",
        "middleName": "Beneficiary_Middle_Name",
        "lastName": "Beneficiary_Last_Name",
        "telephone": "12345678910",
        "sex": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "idType": "0",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "accountNumber": "",
        "sourceIncome": "1",
        "creatTime": "2020-01-07 15:49:35"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号
/entity/beneficiaryNo | String | NextPls收款人唯一编号
/entity/firstName | String | 收款人名
/entity/middleName | String | 收款人中名
/entity/lastName | String | 收款人姓
/entity/telephone | String | 收款人手机号
/entity/email | String | 收款人邮箱
/entity/address1 | String | 收款人地址1
/entity/address2 | String | 收款人地址2
/entity/address3 | String | 收款人地址3
/entity/idType | int | 收款人证件类型
/entity/idNumber | String | 收款人证件号码
/entity/idDesc | String | 收款人描述
/entity/idIssueDate | String | 收款人证件生效时间
/entity/idExpDate | String | 收款人证件失效时间
/entity/birthdate | String | 收款人生日
/entity/sex | String | 收款人性别
/entity/nationality | String | 收款人国籍 
/entity/accountNumber | String | 收款人银行账号
/entity/sourceIncome | String | 收款人收入来源
/entity/creatTime | String | 收款人创建时间
/msg | String | 返回消息

## GetRemitter
获取汇款人信息 
### HTTP Request
<span class="http-method post">POST</span> `/get/remitter`

> Request Body:

```json
{
    "apiName": "GET_REMITTER",
    "entity": {
        "clientRemitterNo": "RE1233112",
        "remitterNo": "RP122141"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称
/entity/clientRemitterNo | String | 客户端汇款人唯一编号
/entity/remitterNo | String | NextPls汇款人唯一编号

> Response Body:

```json
{
    "apiName": "GET_REMITTER_R",
    "code": "200",
    "entity": {
        "clientRemitterNo": "RE1233112",
        "remitterNo": "RP122141",
        "firstName": "REMITTER_First_Name",
        "middleName": "REMITTER_Middle_Name",
        "lastName": "REMITTER_Last_Name",
        "telephone": "12345678910",
        "sex": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "idType": "0",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "accountNumber": "",
        "sourceIncome": "1",
        "creatTime": "2020-01-07 15:49:35"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientRemitterNo | String | 客户端汇款人唯一编号
/entity/remitterNo | String | NextPls汇款人唯一编号
/entity/firstName | String | 汇款人名
/entity/middleName | String | 汇款人中名
/entity/lastName | String | 汇款人姓
/entity/telephone | String | 汇款人手机号
/entity/email | String | 汇款人邮箱
/entity/address1 | String | 汇款人地址1
/entity/address2 | String | 汇款人地址2
/entity/address3 | String | 汇款人地址3
/entity/idType | int | 汇款人证件类型
/entity/idNumber | String | 汇款人证件号码
/entity/idDesc | String | 汇款人描述
/entity/idIssueDate | String | 汇款人证件生效时间
/entity/idExpDate | String | 汇款人证件失效时间
/entity/birthdate | String | 汇款人生日
/entity/sex | String | 汇款人性别
/entity/nationality | String | 汇款人国籍
/entity/accountNumber | String | 汇款人银行账号
/entity/sourceIncome | String | 汇款人收入来源
/entity/creatTime | String | 汇款人创建时间
/msg | String | 返回消息

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

> Request Body:

```json
{
    "apiName": "DO_TRANSACTION_PRE",
    "entity": {
        "payInCurrency": "HKD",
        "payOutCurrency": "PHP",
        "transferCurrency": "HKD",
        "transferAmount": "100",
        "paymentMode": "Bank"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/payInCurrency | String | 存入币种 | M
/entity/payOutCurrency | String | 到账币种 | M
/entity/transferCurrency | String | 结算币种 | M
/entity/transferAmount | String | 结算金额 | M
/entity/paymentMode | String | 支付方式 | M

> Response Body:

```json
{
    "apiName": "GET_REMITTER_R",
    "code": "200",
    "entity": {
        "exchangeRate": "6.580162",
        "commission": "10.00",
        "payInCurrency": "HKD",
        "payoutCurrency": "PHP",
        "payInAmount": "100",
        "payoutAmount": "658.02",
        "totalPayable": "110.80",
        "vatValue": "0.80",
        "vatPercentage": "8",
        "recommendAgent": "75",
        "payoutBranchCode": "1117"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/exchangeRate | String | 汇率
/entity/commission | String | 手续费
/entity/payInCurrency | String | 存入币种
/entity/payoutCurrency | String | 到账币种
/entity/payInAmount | String | 预交金额
/entity/payoutAmount | String | 到账金额
/entity/totalPayable | String | 应支付金额
/entity/vatValue | String | 增值税金额
/entity/vatPercentage | String | 增值税税率
/entity/recommendAgent | String | 
/entity/payoutBranchCode | String | 到账银行代码
/msg | String | 返回消息

## DoTransactionAdd
创建订单 
### HTTP Request
<span class="http-method post">POST</span> `/do/transaction/add`

> Request Body:

```json
{
    "apiName": "DO_TRANSACTION_ADD",
    "entity": {
        "clientTxnNo": "TXN141231",
        "payOutCurrency": "HKD",
        "payOutCountry": "PHP",
        "clientRemitterNo": "RE1233112",
        "remitterNo": "RP122141",
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "RP122141",
        "relationship": "Friend",
        "paymentMode": "BANK",
        "purposeCode": "3",
        "payInCountry": "HKG",
        "payInCurrency": "HKD",
        "settlementCurrency": "HKD"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientTxnNo | String | 客户端订单唯一编号 | M
/entity/payOutCurrency | String | 到账币种 | M
/entity/payOutCountry | String | 到账城市 | M
/entity/clientRemitterNo | String | 客户端汇款人唯一编号 | M
/entity/remitterNo | String | NextPls汇款人唯一编号 | M
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
/entity/beneficiaryNo | String | NextPls收款人唯一编号 | M
/entity/relationship | String | 汇款人与收款人关系 | M
/entity/paymentMode | String | 支付方式 | M
/entity/purposeCode | String | 转账原因 | M
/entity/payInCountry | String | 发起转账城市 | M
/entity/payInCurrency | String | 发起转账币种 | M
/entity/settlementCurrency | String | 结算币种 | M

> Response Body:

```json
{
    "apiName": "DO_TRANSACTION_ADD_R",
    "code": "200",
    "entity": {
        "clientTxnNo": "RE1233112",
        "txnNo": "RP122141"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientTxnNo | String | 客户端订单唯一编号
/entity/txnNo | String | NextPls订单唯一编号
/msg | String | 返回消息

## GetTransactionStatus
检查交易状态
### HTTP Request
<span class="http-method post">POST</span> `/get/transaction/status`

> Request Body:

```json
{
    "apiName": "GET_TRANSACTION_STATUS",
    "entity": {
        "clientTxnNo": "RE1233112",
        "txnNo": "RP122141"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientTxnNo | String | 客户端订单唯一编号 | M
/entity/txnNo | String | NextPls订单唯一编号 | M

> Response Body:

```json
{
    "apiName": "GET_TRANSACTION_STATUS_R",
    "code": "200",
    "entity": {
        "clientTxnNo": "RE1233112",
        "txnNo": "RP122141",
        "status": "INIT"
    },
    "msg": "success"
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientTxnNo | String | 客户端订单唯一编号
/entity/txnNo | String | NextPls订单唯一编号
/entity/status | String | 订单状态
/msg | String | 返回消息

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

> Request Body:

```json
{
    "apiName": "GET_TRANSACTION",
    "entity": {
        "clientTxnNo": "RE1233112",
        "txnNo": "RP122141"
    }
}
```

### Request Body
Pointer | Type | Description | Required
--------- | ------- | ------- | ----------
/apiName | String | 调用接口名称 | M
/entity/clientTxnNo | String | 客户端订单唯一编号 | M
/entity/txnNo | String | NextPls订单唯一编号 | M

> Response Body:

```json
{
    "apiName": "GET_TRANSACTION_R",
    "code": "200",
    "entity": {
        "clientTxnNo": "TXN141231",
        "txnNo": "RP122141",
        "payOutCurrency": "HKD",
        "payOutCountry": "PHP",
        "clientRemitterNo": "RE1233112",
        "remitterNo": "RP122141",
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "RP122145",
        "relationship": "Friend",
        "paymentMode": "BANK",
        "purposeCode": "3",
        "payInCountry": "HKG",
        "payInCurrency": "HKD",
        "settlementCurrency": "HKD",
        "creatTime": "2020-01-07 15:49:35"
    }
}
```

### Response Body
Pointer | Type | Description
--------- | ------- | -----------
/apiName | String | 调用接口名称
/code | String | 返回码
/entity/clientTxnNo | String | 客户端订单唯一编号
/entity/txnNo | String | NextPls订单唯一编号
/entity/payOutCurrency | String | 到账币种
/entity/payOutCountry | String | 到账城市
/entity/clientRemitterNo | String | 客户端汇款人唯一编号
/entity/remitterNo | String | NextPls汇款人唯一编号
/entity/clientBeneficiaryNo | String | 客户端收款人唯一编号
/entity/beneficiaryNo | String | NextPls收款人唯一编号
/entity/relationship | String | 汇款人与收款人关系
/entity/paymentMode | String | 支付方式
/entity/purposeCode | String | 转账原因
/entity/payInCountry | String | 发起转账城市
/entity/payInCurrency | String | 发起转账币种
/entity/settlementCurrency | String | 结算币种
/entity/creatTime | String | 订单创建时间
/msg | String | 返回消息
