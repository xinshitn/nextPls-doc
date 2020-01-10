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
## Cryptography in NextPls API(NextPls API的加密方式)
### Request
#### Generating a CEK(生成一个CEK)
All request body should be encrypted with RSA algorithm before sending to the NextPls server.So it needs a CEK(Content Encryption Key) which will be delivered to NextPls server as described in the Content-Code field in the http header.
(消息在发送到NextPls之前，所有的请求体都应该使用加密算法对请求体进行加密。因此，我们需要一个CEK(内容加密密钥),该密钥将被加密后放在请求头的Content-Code的字段中发送到NextPls服务器。
  
我们希望一个CEK由两个16位字符串拼接而成(16位的初始向量ivParameter和16位的AES密钥sKey)
  
##### 示例
item | ASCII_string 
--------- | -------
sKey | cek_tester_remit
ivParameter | initial_tester01
CEK | cek_tester_remitinitial_tester01
  
<aside class="notice">
CEK不必为有特殊含义的字符串
</aside>
  
#### Encrypting body part with the CEK(利用CEK对方法体进行加密)
在对方法体进行加密时，应使用AES/CBC/PKCS5Padding算法对方法体进行加密，之后应使用BASE64对结果进行转码。

##### 示例
下面的Java代码展示了如何使用AES/CBC/PKCS5Padding算法生成加密体的示例
```
    Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
    byte[] raw = sKey.getBytes();
    SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
    IvParameterSpec iv = new IvParameterSpec(ivParameter.getBytes());
    cipher.init(Cipher.ENCRYPT_MODE, skeySpec, iv);
    byte[] encrypted = cipher.doFinal(body.getBytes("UTF-8"));
    String encryptedBody = Base64.getEncoder().encodeToString(encrypted);
```

#### Encrypting CEK with NextPls public key(利用NextPls公钥对CEK进行加密)
想要请求API调用的代理方必须将加密后的CEK放在请求头之中。应使用RSA加密算法对CEK进行加密，所有的代理方都会获得由NextPls提供的一个用于CEK加密的公钥。

##### 示例
下面的Java代码展示了如何使用RSA加密算法对CEK进行加密
```
    Cipher cipher = Cipher.getInstance("RSA");
    cipher.init(Cipher.ENCRYPT_MODE, publicKey);
    byte[] sbt = source.getBytes(ENCODING);
    byte[] epByte = cipher.doFinal(sbt);
    String encryptedCEK = Base64.getEncoder().encodeToString(epByte);
```
  
#### Generating signature(生成签名)
想要请求API调用的代理方还必须在请求头中附加签名值。签名值用于认证请求是否来自于代理方。
  
签名为对方法体加密结果进行加密，签名算法为SHA256withRSA，之后仍需使用BASE64对结果进行转码。结果放在请求头的Signature的字段中。

##### 示例
下面的Java代码展示了如何使用签名如何生成
```
    byte[] signed = signSHA256withRSA(encryptedBody);
    String signBase64 = Base64.getEncoder().encodeToString(signed);
    String signature = StringUtils.deleteWhitespace(signBase64);
```
  
#### Generating request header and body(生成请求头和请求体)
使用加密的CEK、加密的方法体和签名便可组成一个完整的请求
  
##### 示例
>Header
>>Accept:application/base64
>>Authorization:AU1234567
>>Content-Code:dpcnD2YnJpCr0EIUNkz5so9rsBloBJRk/ie/awlgOV8ECoob3yB9QoAtP7LQNtOmziQOrFXueD0B62gc2f+AqCOs5D66sWQjlCWnjyqOkFjfow93CtYvGwpuoiqNArk39qvbe3lTo2g3qalg5YEe0Reohf2yHFbsYrwtgvBWQUI=
>>Signature:CiELHoMm9ab9FduNLWQZ15OI4O+ms8xGjJBEMyRsLRvi9cQI0QctIajwnejCrri7q0ep2zf6Fqn9DIWpF2vgsAHK3SWr2SkykxY2Sheisv1hjpxOVYgnLDcI/0KXTFSjHQWQ+JG6d8VvwrAErqY0jk9pafUj1SfxrHpUzEUWDGQ=

>Body
>Deo7f9su8hdo0PCCKxyjuCRVCKAotP01jgfDJd82jrLQAvEyXK+hwNMF2mLKidCERaS604yzdQ2REQ0Rja/2H87VLLmsQx7Bkbe0yah8ALIaCabwY30aG/FPsjY4Y7OhujaEAzOVRUrV21iYDL5nUg=

### Response
#### Verifying signature(验证签名)
为了验证NextPls服务器的真实性，代理方需要将签名进行SHA256withRSA算法解密，并与加密请求体进行校验

##### 示例
```
    Signature signetcheck = Signature.getInstance(SIGNATURE_ALGORITHM);
    signetcheck.initVerify(getPublicKey());
    signetcheck.update(data.getBytes(ENCODING));
    boolean result = signetcheck.verify(Base64.getDecoder().decode(sign));
```

#### Deriving CEK

#### Decrypting body part with CEK
## Keys
在下面的文档中缩写值的定义

参数 | 类型 | 描述
--------- | ------- | -----------
M | string | 必填字段
O | string | 选填字段
C | string | 有前置条件的选填或必填字段

# 基础信息
## GetCountryPayMode
获取指定国家支持的支付模式 
### HTTP Request
<span class="http-method post">POST</span> `/get/countryPayMode`

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------

### Response Body
参数 | | 类型 | 描述
--------- | ------- | ------- |-----------

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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:“
    -H "Signature:"
    -H "Content-Code:"
    -d
    '{
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
     }'
```

### Request Body
参数 | 参数 | 类型 | 描述 | O/M
--------- | :------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
| | firstName | String | 收款人名 | M
| | middleName | String | 收款人中名 | O
| | lastName | String | 收款人姓 | M
| | telephone | String | 收款人手机号 | M
| | email | String | 收款人邮箱 | O
| | address1 | String | 收款人地址1 | M
| | address2 | String | 收款人地址2 | O
| | address3 | String | 收款人地址3 | O
| | idType | int | 收款人证件类型 | M
| | idNumber | String | 收款人证件号码 | M
| | idDesc | String | 收款人描述 | O
| | idIssueDate | String | 收款人证件生效时间 | O
| | idExpDate | String | 收款人证件失效时间 | O
| | birthdate | String | 收款人生日 | O
| | sex | String | 收款人性别 | O
| | nationality | String | 收款人国籍 | M
| | bankCode | String | 收款人账户银行编号 | M
| | bankAccountNumber | String | 收款人银行账户 | M
| | bankAccountName | String | 收款人账户银行名称 | M
| | bankAddress | String | 收款人账户银行地址 | O

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName |  | String | 调用接口名称
code |  |String | 返回码
entity |
| | clientBeneficiaryNo | String | 客户端收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
msg |  |String | 返回消息


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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| |clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
| | beneficiaryNo | String | NextPls收款人唯一编号 | M
| | firstName | String | 收款人名 | O
| | middleName | String | 收款人中名 | O
| | lastName | String | 收款人姓 | O
| | telephone | String | 收款人手机号 | O
| | email | String | 收款人邮箱 | O
| | address1 | String | 收款人地址1 | O
| | address2 | String | 收款人地址2 | O
| | address3 | String | 收款人地址3 | O
| | idType | int | 收款人证件类型 | O
| | idNumber | String | 收款人证件号码 | O
| | idDesc | String | 收款人描述 | O
| | idIssueDate | String | 收款人证件生效时间 | O
| | idExpDate | String | 收款人证件失效时间 | O
| | birthdate | String | 收款人生日 | O
| | sex | String | 收款人性别 | O
| | nationality | String | 收款人国籍 | O
| | bankCode | String | 收款人账户银行编号 | O
| | bankAccountNumber | String | 收款人银行账户 | O
| | bankAccountName | String | 收款人账户银行名称 | O
| | bankAddress | String | 收款人账户银行地址 | O

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientBeneficiaryNo | String | 客户端收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
msg | | String | 返回消息

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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientRemitterNo | String | 客户端汇款人唯一编号 | M
| | firstName | String | 汇款人名 | M
| | middleName | String | 汇款人中名 | O
| | lastName | String | 汇款人姓 | M
| | telephone | String | 汇款人手机号 | M
| | email | String | 汇款人邮箱 | O
| | address1 | String | 汇款人地址1 | M
| | address2 | String | 汇款人地址2 | O
| | address3 | String | 汇款人地址3 | O
| | idType | int | 汇款人证件类型 | M
| | idNumber | String | 汇款人证件号码 | M
| | idDesc | String | 汇款人描述 | O
| | idIssueDate | String | 汇款人证件生效时间 | O
| | idExpDate | String | 汇款人证件失效时间 | O
| | birthdate | String | 汇款人生日 | O
| | sex | String | 汇款人性别 | O
| | nationality | String | 汇款人国籍 | M
| | accountNumber | String | 汇款人银行账号 | O
| | sourceIncome | String | 汇款人收入来源 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientRemitterNo | String | 客户端汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号
msg | | String | 返回消息

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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientRemitterNo | String | 客户端汇款人唯一编号 | M
| | remitterNo | String | NextPls汇款人唯一编号 | M
| | firstName | String | 汇款人名 | O
| | middleName | String | 汇款人中名 | O
| | lastName | String | 汇款人姓 | O
| | telephone | String | 汇款人手机号 | O
| | email | String | 汇款人邮箱 | O
| | address1 | String | 汇款人地址1 | O
| | address2 | String | 汇款人地址2 | O
| | address3 | String | 汇款人地址3 | O
| | idType | int | 汇款人证件类型 | O
| | idNumber | String | 汇款人证件号码 | O
| | idDesc | String | 汇款人描述 | O
| | idIssueDate | String | 汇款人证件生效时间 | O
| | idExpDate | String | 汇款人证件失效时间 | O
| | birthdate | String | 汇款人生日 | O
| | sex | String | 汇款人性别 | O
| | nationality | String | 汇款人国籍 | O
| | accountNumber | String | 汇款人银行账号 | O
| | sourceIncome | String | 汇款人收入来源 | O

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientRemitterNo | String | 客户端汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号
msg | | String | 返回消息


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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
| | BeneficiaryNo | String | NextPls收款人唯一编号 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientBeneficiaryNo | String | 客户端收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
msg | | String | 返回消息

## GetBeneficiaryList
获取收款人列表
### HTTP Request
<span class="http-method post">POST</span> `/get/beneficiary/list`

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -----

### Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------

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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
| | BeneficiaryNo | String | NextPls收款人唯一编号 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientBeneficiaryNo | String | 客户端收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
| | firstName | String | 收款人名
| | middleName | String | 收款人中名
| | lastName | String | 收款人姓
| | telephone | String | 收款人手机号
| | email | String | 收款人邮箱
| | address1 | String | 收款人地址1
| | address2 | String | 收款人地址2
| | address3 | String | 收款人地址3
| | idType | int | 收款人证件类型
| | idNumber | String | 收款人证件号码
| | idDesc | String | 收款人描述
| | idIssueDate | String | 收款人证件生效时间
| | idExpDate | String | 收款人证件失效时间
| | birthdate | String | 收款人生日
| | sex | String | 收款人性别
| | nationality | String | 收款人国籍 
| | accountNumber | String | 收款人银行账号
| | sourceIncome | String | 收款人收入来源
| | creatTime | String | 收款人创建时间
msg | | String | 返回消息

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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称
entity |
| | clientRemitterNo | String | 客户端汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientRemitterNo | String | 客户端汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号
| | firstName | String | 汇款人名
| | middleName | String | 汇款人中名
| | lastName | String | 汇款人姓
| | telephone | String | 汇款人手机号
| | email | String | 汇款人邮箱
| | address1 | String | 汇款人地址1
| | address2 | String | 汇款人地址2
| | address3 | String | 汇款人地址3
| | idType | int | 汇款人证件类型
| | idNumber | String | 汇款人证件号码
| | idDesc | String | 汇款人描述
| | idIssueDate | String | 汇款人证件生效时间
| | idExpDate | String | 汇款人证件失效时间
| | birthdate | String | 汇款人生日
| | sex | String | 汇款人性别
| | nationality | String | 汇款人国籍
| | accountNumber | String | 汇款人银行账号
| | sourceIncome | String | 汇款人收入来源
| | creatTime | String | 汇款人创建时间
msg | | String | 返回消息

# 交易
## GetBalance
获取账户余额 
### HTTP Request
<span class="http-method post">POST</span> `/get/balance`

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------

### Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------

## GetExRate
获取汇率 
### HTTP Request
<span class="http-method post">POST</span> `/get/exRate`

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------

### Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------

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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | payInCurrency | String | 存入币种 | M
| | payOutCurrency | String | 到账币种 | M
| | transferCurrency | String | 结算币种 | M
| | transferAmount | String | 结算金额 | M
| | paymentMode | String | 支付方式 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | exchangeRate | String | 汇率
| | commission | String | 手续费
| | payInCurrency | String | 存入币种
| | payoutCurrency | String | 到账币种
| | payInAmount | String | 预交金额
| | payoutAmount | String | 到账金额
| | totalPayable | String | 应支付金额
| | vatValue | String | 增值税金额
| | vatPercentage | String | 增值税税率
| | recommendAgent | String | 
| | payoutBranchCode | String | 到账银行代码
msg | | String | 返回消息

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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientTxnNo | String | 客户端订单唯一编号 | M
| | payOutCurrency | String | 到账币种 | M
| | payOutCountry | String | 到账城市 | M
| | clientRemitterNo | String | 客户端汇款人唯一编号 | M
| | remitterNo | String | NextPls汇款人唯一编号 | M
| | clientBeneficiaryNo | String | 客户端收款人唯一编号 | M
| | beneficiaryNo | String | NextPls收款人唯一编号 | M
| | relationship | String | 汇款人与收款人关系 | M
| | paymentMode | String | 支付方式 | M
| | purposeCode | String | 转账原因 | M
| | payInCountry | String | 发起转账城市 | M
| | payInCurrency | String | 发起转账币种 | M
| | settlementCurrency | String | 结算币种 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientTxnNo | String | 客户端订单唯一编号
| | txnNo | String | NextPls订单唯一编号
msg | | String | 返回消息

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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientTxnNo | String | 客户端订单唯一编号 | M
| | txnNo | String | NextPls订单唯一编号 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity |
| | clientTxnNo | String | 客户端订单唯一编号
| | txnNo | String | NextPls订单唯一编号
| | status | String | 订单状态
msg | | String | 返回消息

## GetTransactionList
获取交易列表
### HTTP Request
<span class="http-method post">POST</span> `/get/transaction/list`

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------

### Response Body
参数 | Type | Description
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
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity |
| | clientTxnNo | String | 客户端订单唯一编号 | M
| | txnNo | String | NextPls订单唯一编号 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName |  | String | 调用接口名称
code |  | String | 返回码
entity |
| | clientTxnNo | String | 客户端订单唯一编号
| | txnNo | String | NextPls订单唯一编号
| | payOutCurrency | String | 到账币种
| | payOutCountry | String | 到账城市
| | clientRemitterNo | String | 客户端汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号
| | clientBeneficiaryNo | String | 客户端收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
| | relationship | String | 汇款人与收款人关系
| | paymentMode | String | 支付方式
| | purposeCode | String | 转账原因
| | payInCountry | String | 发起转账城市
| | payInCurrency | String | 发起转账币种
| | settlementCurrency | String | 结算币种
| | creatTime | String | 订单创建时间
msg | | String | 返回消息
