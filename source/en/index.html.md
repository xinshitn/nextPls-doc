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
## Cryptography in QSRemit API 
### Request
1. Generating a CEK

All request body should be encrypted with RSA algorithm before sending to the NextPls server.So it needs a CEK(Content Encryption Key) which will be delivered to NextPls server as described in the Content-Code field in the http header.
  
Agent who wants to make a request API should generate an AES-128 key for the CEK,which is comprised of 32 bytes random digits (16 bytes for initial vectors and 16 bytes for AES key). 
  
### example
item | ASCII_string 
--------- | -------
sKey | cek_tester_remit
ivParameter | initial_tester01
CEK | cek_tester_remitinitial_tester01
  
<aside class="notice">
CEK is not a necessarily human readable ASCII string
</aside>
  
2. Encrypting body part with the CEK

Agent who wants to make a request API should encrypt the body part with CEK before sending to the NextPls server

在对方法体进行加密时，应使用AES/CBC/PKCS5Padding算法对方法体进行加密，之后应使用BASE64对结果进行转码。

>加密体加密示例:

```java
    public class example{
        public static void main(String[] args){
          
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            byte[] raw = sKey.getBytes();
            SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
            IvParameterSpec iv = new IvParameterSpec(ivParameter.getBytes());
            cipher.init(Cipher.ENCRYPT_MODE, skeySpec, iv);
            byte[] encrypted = cipher.doFinal(body.getBytes("UTF-8"));
            String encryptedBody = Base64.getEncoder().encodeToString(encrypted);
            System.out.println("加密后的方法体为:" + encryptedBody);
            
        }
    }
```

2. Encrypting CEK with NextPls public key

想要请求API调用的代理方必须将加密后的CEK放在请求头之中。应使用RSA加密算法对CEK进行加密，所有的代理方都会获得由NextPls提供的一个用于CEK加密的公钥。

>CEK加密示例：

```java
    public class example{
        public static void main(String[] args){
          
            Cipher cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
            byte[] sbt = source.getBytes(ENCODING);
            byte[] epByte = cipher.doFinal(sbt);
            String encryptedCEK = Base64.getEncoder().encodeToString(epByte);
            System.out.println("加密后的CEK值为:" + encryptedCEK);
            
        }
    }
```
  
3. Generating signature

想要请求API调用的代理方还必须在请求头中附加签名值。签名值用于认证请求是否来自于代理方。
  
签名为对方法体加密结果进行加密，签名算法为SHA256withRSA，之后仍需使用BASE64对结果进行转码。结果放在请求头的Signature的字段中。

>签名生成示例：

```java
    public class example{
        public static void main(String[] args){
          
            Signature sign = Signature.getInstance("SHA256withRSA", new BouncyCastleProvider());
            sign.initSign(getPrivateKey());
            sign.update(plainText.getBytes(ENCODING));
            byte[] signed = sign.sign();
            String signBase64 = Base64.getEncoder().encodeToString(signed);
            String signature = StringUtils.deleteWhitespace(signBase64);
            System.out.println("签名值为:" + signature);
            
        }
    }
```
  
4. Generating request header and body

使用加密的CEK、加密的方法体和签名便可组成一个完整的请求
  
### 示例

Header |
-------- |
Accept : application/base64 |
Authorization : AU1234567 |
Content-Code : dpcnD2YnJpCr0EIUNkz5so9rsBloBJRk/ie/awlgOV8ECoob3yB9QoAtP7LQNtOmziQOrFXueD0B62gc2f+AqCOs5D66sWQjlCWnjyqOkFjfow93CtYvGwpuoiqNArk39qvbe3lTo2g3qalg5YEe0Reohf2yHFbsYrwtgvBWQUI= |
Signature : CiELHoMm9ab9FduNLWQZ15OI4O+ms8xGjJBEMyRsLRvi9cQI0QctIajwnejCrri7q0ep2zf6Fqn9DIWpF2vgsAHK3SWr2SkykxY2Sheisv1hjpxOVYgnLDcI/0KXTFSjHQWQ+JG6d8VvwrAErqY0jk9pafUj1SfxrHpUzEUWDGQ= |

Body |
------ |
Deo7f9su8hdo0PCCKxyjuCRVCKAotP01jgfDJd82jrLQAvEyXK+hwNMF2mLKidCERaS604yzdQ2REQ0Rja/2H87VLLmsQx7Bkbe0yah8ALIaCabwY30aG/FPsjY4Y7OhujaEAzOVRUrV21iYDL5nUg= |

### Response
1. Verifying signature

为了验证NextPls服务器的真实性，代理方需要将签名进行SHA256withRSA算法解密，并与加密请求体进行校验,如果结果不是true，这意味着无效的签名，响应主体的真实性得不到保证，合作伙伴不应该进行进一步的处理

>验证签名示例：

```java
    public class example{
        public static void main(String[] args){
          
            Signature signetcheck = Signature.getInstance(SIGNATURE_ALGORITHM);
            signetcheck.initVerify(getPublicKey());
            signetcheck.update(data.getBytes(ENCODING));
            boolean result = signetcheck.verify(Base64.getDecoder().decode(sign));
            System.out.println("签名是否通过验证：" + result);
            
        }
    }
```

2. Deriving CEK

所有的响应体在从NextPls服务器发送之前都使用AES/CBC/PKCS5Padding算法进行加密。因此，它需要首先从http报头中的Content-Code字段中解析CEK(通过内容加密密钥)

CEK由32字节的随机字符(初始向量为16字节，AES密钥为16字节)组成。它是用代理方的公钥(RSA)进行加密的。所以代理方需要准备好相应的私钥。

CEK可以从响应头中的Content-Code字段中获得

>获取CEK示例

```java
    public class example{
        public static void main(String[] args){
            
            //解密块最大长度
            private int maxDecryptBlock = 256;
            byte[] encryptedData = Base64.getDecoder().decode(encryptText);
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
            cipher.init(Cipher.DECRYPT_MODE, getPrivateKey());
            int inputLen = encryptedData.length;
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int offSet = 0;
            byte[] cache;
            int i = 0;
            // 对数据分段解密
            while (inputLen - offSet > 0) {
                if (inputLen - offSet > maxDecryptBlock) {
                    cache = cipher.doFinal(encryptedData, offSet, maxDecryptBlock);
                } else {
                    cache = cipher.doFinal(encryptedData, offSet, inputLen - offSet);
                }
                out.write(cache, 0, cache.length);
                i++;
                offSet = i * maxDecryptBlock;
            }
            byte[] decryptedData = out.toByteArray();
            out.close();
            String CEK = new String(decryptedData);
            System.out.println("解析获得CEK为：" + CEK);
          
        }
    }
```

3. Decrypting body part with CEK

代理方可以在响应体中通过获取的CEK获取响应的json对象

>获取响应数据示例

```java
    public class example{
        public static void main(String[] args){
            
            byte[] raw = sKey.getBytes("ASCII");
            SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            IvParameterSpec iv = new IvParameterSpec(ivParameter.getBytes());
            cipher.init(Cipher.DECRYPT_MODE, skeySpec, iv);
            //先用base64解密
            byte[] encrypted1 = Base64.getDecoder().decode(sSrc);
            byte[] original = cipher.doFinal(encrypted1);
            String encodingFormat = "UTF-8";
            String resBody = new String(original, encodingFormat);
            System.out.println("解析获得响应数据为：" + resBody);
          
        }
    }
```

## Keys

Definition of abbreviated values found in the documentation.

Parameter | Type | Description
--------- | ------- | -----------
M | string | value for this field is required
O | string | value for this field is optional
C | string | The requirement for this field is conditional based on other field values

# Basic Information
## GetCountryPayMode
获取指定国家支持的支付模式 
### HTTP Request
<span class="http-method post">POST</span> `/get/countryPayMode`

The function is not open yet, Coming soon!

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
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
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
Parameter | | Type | Description | O/M
--------- | :------- | ------- | ---------- | -------
apiName | | String | the name of the invocation interface | M
entity | | Object | agent request parameter list | M
| | clientBeneficiaryNo | String | Unique code for agent beneficiary | M
| | firstName | String | Beneficiary First Name | M
| | middleName | String | Beneficiary Middle Name | O
| | lastName | String | Beneficiary Last Name | M
| | telephone | String | Telephone Number of Beneficiary | M
| | email | String | Email of Beneficiary | O
| | address1 | String | Beneficiary Address line 1 | M
| | address2 | String | Beneficiary Address line 2 | O
| | address3 | String | 收款人地址3 | O
| | idType | int | 收款人证件类型 | M
| | idNumber | String | 收款人证件号码 | M
| | idDesc | String | 收款人Description | O
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName |  | String | 调用接口名称
code |  |String | 返回码
entity | | Object | NextPls返回结果
| | clientBeneficiaryNo | String | 代理方收款人唯一编号
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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
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
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| |clientBeneficiaryNo | String | 代理方收款人唯一编号 | M
| | beneficiaryNo | String | NextPls收款人唯一编号 | M
| | firstName | String | Beneficiary First Name | O
| | middleName | String | Beneficiary Middle Name | O
| | lastName | String | Beneficiary Last Name | O
| | telephone | String | Telephone Number of Beneficiary | O
| | email | String | Email of Beneficiary | O
| | address1 | String | Beneficiary Address line 1 | O
| | address2 | String | Beneficiary Address line 2 | O
| | address3 | String | Beneficiary Address line 3 | O
| | idType | int | 收款人证件类型 | O
| | idNumber | String | 收款人证件号码 | O
| | idDesc | String | 收款人Description | O
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientBeneficiaryNo | String | 代理方收款人唯一编号
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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
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
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| | clientRemitterNo | String | 代理方汇款人唯一编号 | M
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
| | idDesc | String | 汇款人Description | O
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientRemitterNo | String | 代理方汇款人唯一编号
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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
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
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| | clientRemitterNo | String | 代理方汇款人唯一编号 | M
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
| | idDesc | String | 汇款人Description | O
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientRemitterNo | String | 代理方汇款人唯一编号
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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "DO_BENEFICIARY_DEL",
         "entity": {
             "clientBeneficiaryNo": "BE1233112",
             "beneficiaryNo": "RP122141"
         }
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| | clientBeneficiaryNo | String | 代理方收款人唯一编号 | M
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientBeneficiaryNo | String | 代理方收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
msg | | String | 返回消息

## GetBeneficiaryList
获取收款人列表
### HTTP Request
<span class="http-method post">POST</span> `/get/beneficiary/list`

The function is not open yet, Coming soon!

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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
        "apiName": "GET_BENEFICIARY",
        "entity": {
            "clientBeneficiaryNo": "BE1233112",
            "beneficiaryNo": "RP122141"
        } 
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| | clientBeneficiaryNo | String | 代理方收款人唯一编号 | M
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientBeneficiaryNo | String | 代理方收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
| | firstName | String | Beneficiary First Name
| | middleName | String | Beneficiary Middle Name
| | lastName | String | Beneficiary Last Name
| | telephone | String | Telephone Number of Beneficiary
| | email | String | Email of Beneficiary
| | address1 | String | Beneficiary Address line 1
| | address2 | String | Beneficiary Address line 2
| | address3 | String | Beneficiary Address line 3
| | idType | int | 收款人证件类型
| | idNumber | String | 收款人证件号码
| | idDesc | String | 收款人Description
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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "GET_REMITTER",
         "entity": {
             "clientRemitterNo": "RE1233112",
             "remitterNo": "RP122141"
         }
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称
entity | | Object | 代理方请求参数 | M
| | clientRemitterNo | String | 代理方汇款人唯一编号
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientRemitterNo | String | 代理方汇款人唯一编号
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
| | idDesc | String | 汇款人Description
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

The function is not open yet, Coming soon!

## GetExRate
获取汇率 
### HTTP Request
<span class="http-method post">POST</span> `/get/exRate`

The function is not open yet, Coming soon!

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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "DO_TRANSACTION_PRE",
         "entity": {
             "payInCurrency": "HKD",
             "payOutCurrency": "PHP",
             "transferCurrency": "HKD",
             "transferAmount": "100",
             "paymentMode": "Bank"
         }
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
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
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| | clientTxnNo | String | 代理方订单唯一编号 | M
| | payOutCurrency | String | 到账币种 | M
| | payOutCountry | String | 到账城市 | M
| | clientRemitterNo | String | 代理方汇款人唯一编号 | M
| | remitterNo | String | NextPls汇款人唯一编号 | M
| | clientBeneficiaryNo | String | 代理方收款人唯一编号 | M
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
Parameter |  | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientTxnNo | String | 代理方订单唯一编号
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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "GET_TRANSACTION_STATUS",
         "entity": {
             "clientTxnNo": "RE1233112",
             "txnNo": "RP122141"
         }
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| | clientTxnNo | String | 代理方订单唯一编号 | M
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
Parameter |  | Type | Description
--------- | ------- | ------- |-----------
apiName | | String | 调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientTxnNo | String | 代理方订单唯一编号
| | txnNo | String | NextPls订单唯一编号
| | status | String | 订单状态
msg | | String | 返回消息

## GetTransactionList
获取交易列表
### HTTP Request
<span class="http-method post">POST</span> `/get/transaction/list`

The function is not open yet, Coming soon!

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
```shell
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "GET_TRANSACTION",
         "entity": {
             "clientTxnNo": "RE1233112",
             "txnNo": "RP122141"
         }
     }'
```

### Request Body
Parameter |  | Type | Description | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 代理方请求参数 | M
| | clientTxnNo | String | 代理方订单唯一编号 | M
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
Parameter |   | Type | Description
--------- | ------- | ------- |-----------
apiName |  | String | 调用接口名称
code |  | String | 返回码
entity | | Object | NextPls返回结果
| | clientTxnNo | String | 代理方订单唯一编号
| | txnNo | String | NextPls订单唯一编号
| | payOutCurrency | String | 到账币种
| | payOutCountry | String | 到账城市
| | clientRemitterNo | String | 代理方汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号
| | clientBeneficiaryNo | String | 代理方收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
| | relationship | String | 汇款人与收款人关系
| | paymentMode | String | 支付方式
| | purposeCode | String | 转账原因
| | payInCountry | String | 发起转账城市
| | payInCurrency | String | 发起转账币种
| | settlementCurrency | String | 结算币种
| | creatTime | String | 订单创建时间
msg | | String | 返回消息

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
