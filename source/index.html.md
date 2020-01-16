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
## NextPls API的加密方式
### Request
1. 生成一个CEK

消息在发送到NextPls之前，所有的请求体都应该使用加密算法对请求体进行加密。因此，我们需要一个CEK(内容加密密钥),该密钥将被加密后放在请求头的Content-Code的字段中发送到NextPls服务器。
  
我们希望一个CEK由两个16位字符串拼接而成(16位的初始向量ivParameter和16位的AES密钥sKey)
  
### 示例
item | ASCII_string 
--------- | -------
sKey | cek_tester_remit
ivParameter | initial_tester01
CEK | cek_tester_remitinitial_tester01
  
<aside class="notice">
CEK不需要有特殊含义，可以为随机的字符串
</aside>
  
2. 利用CEK对方法体进行加密

在将请求发送到NextPls之前，应使用CEK加密消息体部分

在对消息体进行加密时，应使用AES/CBC/PKCS5Padding算法，之后再使用BASE64对结果进行转码。

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

2. 利用NextPls公钥对CEK进行加密

在调用API前，必须将加密后的CEK放在请求头之中。应使用RSA算法对CEK进行加密，所有的客户都会获得由NextPls提供的一个用于CEK加密的专有公钥。

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

3. 生成签名

在调用API前，还必须在请求头中附加签名值。签名值用于认证请求是否来自于客户方。
  
签名为对“消息体加密后的结果”进行再签名，签名算法为SHA256withRSA，之后仍需使用BASE64对结果进行转码。结果放在请求头的Signature的字段中。

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
  
4. 生成请求头和请求体

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
1. 验证签名

为了验证NextPls服务器的真实性，客户方需要将签名进行SHA256withRSA算法解密，并与加密请求体进行校验,如果结果不是true，这意味着无效的签名，响应主体的真实性得不到保证，客户方不应该进行进一步的处理

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

2. CEK的获取

所有的响应体在从NextPls服务器发送之前都使用AES/CBC/PKCS5Padding算法进行加密。因此，它需要首先从http报头中的Content-Code字段中解析CEK(通过内容加密密钥)

CEK由32字节的随机字符(初始向量为16字节，AES密钥为16字节)组成。它是用客户提供的公钥(RSA)进行加密的。所以客户需要准备好相应的私钥。

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

3. 用CEK解密响应体

您可以在响应体中通过获取的CEK解码，以获取响应的json对象

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
<span class="http-method post">POST</span> `GET_COUNTRY_PAY_MODE`

接口暂未开放，敬请期待！

# 收汇款人信息

## DoRemitterAdd
添加汇款人信息
### HTTP Request
<span class="http-method post">POST</span> `DO_REMITTER_ADD`

> Request Body:

```json
{
    "apiName": "DO_REMITTER_ADD",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "firstName": "Remitter_First_Name",
        "middleName": "Remitter_Middle_Name",
        "lastName": "Remitter_Last_Name",
        "mobile": "12345678910",
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
             "clientRemitterNo": "TEST_B001",
             "firstName": "Remitter_First_Name",
             "middleName": "Remitter_Middle_Name",
             "lastName": "Remitter_Last_Name",
             "mobile": "12345678910",
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
```java
    public class example{
        public static void main(String[] args){
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsRemitterRequestDto remitterRequestDto = new NextPlsRemitterRequestDto();
            remitterRequestDto.setClientRemitterNo("TEST_R001");
            // ...
            NextPlsDoRemitterAddRequest remitterAddRequest = NextPlsDoRemitterAddRequest.build(remitterRequestDto);
            client.execute(remitterAddRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientRemitterNo(20) | String | 客户方汇款人唯一编号 | M
| | firstName(50) | String | 汇款人名 | M
| | middleName(50) | String | 汇款人中名 | O
| | lastName(50) | String | 汇款人姓 | M
| | mobile(20) | String | 汇款人手机号 | M
| | email(50) | String | 汇款人邮箱 | O
| | address1(35) | String | 汇款人地址1 | M
| | address2(35) | String | 汇款人地址2 | O
| | address3(50) | String | 汇款人地址3 | O
| | idType(3) | int | 汇款人证件类型 | M
| | idNumber(20) | String | 汇款人证件号码 | M
| | idDesc(30) | String | 汇款人描述 | O
| | idIssueDate(10) | String | 汇款人证件生效时间 | O
| | idExpDate(10) | String | 汇款人证件失效时间 | O
| | birthdate(10) | String | 汇款人生日 | O
| | sex(1) | String | 汇款人性别 | O
| | nationality(3) | String | 汇款人国籍 | M
| | accountNumber(30) | String | 汇款人银行账号 | O
| | sourceIncome(2) | String | 汇款人收入来源 | M

> Response Body:

```json
{
    "apiName": "DO_REMITTER_ADD",
    "code": "200",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873"
    },
    "msg": "success"
}
```

### Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientRemitterNo | String | 客户方汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号
msg | | String | 返回消息

## DoRemitterEdit
修改汇款人信息
### HTTP Request
<span class="http-method post">POST</span> `DO_REMITTER_EDIT`

> Request Body:

```json
{
    "apiName": "DO_REMITTER_EDIT",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873",
        "firstName": "Remitter_First_Name",
        "middleName": "Remitter_Middle_Name",
        "lastName": "Remitter_Last_Name",
        "mobile": "12345678910",
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
             "clientRemitterNo": "TEST_B001",
             "remitterNo": "JJ201A1131599873",
             "firstName": "Remitter_First_Name",
             "middleName": "Remitter_Middle_Name",
             "lastName": "Remitter_Last_Name",
             "mobile": "12345678910",
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
```java
    public class example{
        public static void main(String[] args){
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsRemitterRequestDto remitterRequestDto = new NextPlsRemitterRequestDto();
            remitterRequestDto.setClientRemitterNo("TEST_R001");
            // ...
            NextPlsDoRemitterEditRequest remitterEditRequest = NextPlsDoRemitterEditRequest.build(remitterRequestDto);
            client.execute(remitterEditRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientRemitterNo(20) | String | 客户方汇款人唯一编号 | M
| | remitterNo(20) | String | NextPls汇款人唯一编号 | M
| | firstName(50) | String | 汇款人名 | O
| | middleName(50) | String | 汇款人中名 | O
| | lastName(50) | String | 汇款人姓 | O
| | mobile(20) | String | 汇款人手机号 | O
| | email(50) | String | 汇款人邮箱 | O
| | address1(35) | String | 汇款人地址1 | O
| | address2(35) | String | 汇款人地址2 | O
| | address3(50) | String | 汇款人地址3 | O
| | idType(3) | int | 汇款人证件类型 | O
| | idNumber(20) | String | 汇款人证件号码 | O
| | idDesc(30) | String | 汇款人描述 | O
| | idIssueDate(10) | String | 汇款人证件生效时间 | O
| | idExpDate(10) | String | 汇款人证件失效时间 | O
| | birthdate(10) | String | 汇款人生日 | O
| | sex(1) | String | 汇款人性别 | O
| | nationality(3) | String | 汇款人国籍 | O
| | accountNumber(30) | String | 汇款人银行账号 | O
| | sourceIncome(2) | String | 汇款人收入来源 | O

> Response Body:

```json
{
    "apiName": "DO_REMITTER_EDIT",
    "code": "200",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873"
    },
    "msg": "success"
}
```

### Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientRemitterNo | String | 客户方汇款人唯一编号
| | remitterNo | String | NextPls汇款人唯一编号
msg | | String | 返回消息

## DoBeneficiaryAdd
添加收款人信息
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
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsBeneficiaryRequestDto beneficiaryRequestDto = new NextPlsBeneficiaryRequestDto();
            beneficiaryRequestDto.setClientBeneficiaryNo("TEST_B001");
            // ...
            NextPlsDoBeneficiaryAddRequest beneficiaryAddRequest = NextPlsDoBeneficiaryAddRequest.build(beneficiaryRequestDto);
            client.execute(beneficiaryAddRequest);
          
        }
    }
```

### Request Body
参数 | | 类型 | 描述 | O/M
--------- | :------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientBeneficiaryNo(20) | String | 客户方收款人唯一编号 | M
| | firstName(50) | String | 收款人名 | M
| | middleName(50) | String | 收款人中名 | O
| | lastName(50) | String | 收款人姓 | M
| | mobile(20) | String | 收款人手机号 | M
| | email(50) | String | 收款人邮箱 | O
| | address1(35) | String | 收款人地址1 | M
| | address2(35) | String | 收款人地址2 | O
| | address3(35) | String | 收款人地址3 | O
| | idType(2) | int | 收款人证件类型 | M
| | idNumber(20) | String | 收款人证件号码 | M
| | idDesc(20) | String | 收款人证件描述，当证件类型为6的时候为必填项 | C
| | idIssueDate(10) | String | 收款人证件生效时间 | O
| | idExpDate(10) | String | 收款人证件失效时间 | O
| | birthdate(10) | String | 收款人生日 | O
| | sex(1) | String | 收款人性别 | O
| | nationality(3) | String | 收款人国籍(3位的国家ISO代码) | M
| | bankCode(20) | String | 收款人账户银行编号 | M
| | bankAccountNumber(30) | String | 收款人银行账户 | M
| | bankAccountName(35) | String | 收款人账户银行名称 | M
| | bankAddress(35) | String | 收款人账户银行地址 | O

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName |  | String | 被调用接口名称
code |  |String | 返回码
entity | | Object | NextPls返回结果
| | clientBeneficiaryNo | String | 客户方收款人唯一编号
| | beneficiaryNo | String | NextPls收款人唯一编号
msg |  |String | 返回消息


## DoBeneficiaryEdit
修改收款人信息
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
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsBeneficiaryRequestDto beneficiaryRequestDto = new NextPlsBeneficiaryRequestDto();
            beneficiaryRequestDto.setClientBeneficiaryNo("TEST_B001");
            // ...
            NextPlsDoBeneficiaryEditRequest beneficiaryEditRequest = NextPlsDoBeneficiaryEditRequest.build(beneficiaryRequestDto);
            client.execute(beneficiaryEditRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| |clientBeneficiaryNo(20) | String | 客户方收款人唯一编号 | M
| | beneficiaryNo(20) | String | NextPls收款人唯一编号 | M
| | firstName(50) | String | 收款人名 | O
| | middleName(50) | String | 收款人中名 | O
| | lastName(50) | String | 收款人姓 | O
| | mobile(20) | String | 收款人手机号 | O
| | email(35) | String | 收款人邮箱 | O
| | address1(35) | String | 收款人地址1 | O
| | address2(35) | String | 收款人地址2 | O
| | address3(35) | String | 收款人地址3 | O
| | idType(2) | int | 收款人证件类型 | O
| | idNumber(20) | String | 收款人证件号码 | O
| | idDesc(20) | String | 收款人描述 | O
| | idIssueDate(10) | String | 收款人证件生效时间 | O
| | idExpDate(10) | String | 收款人证件失效时间 | O
| | birthdate(10) | String | 收款人生日 | O
| | sex(1) | String | 收款人性别 | O
| | nationality(3) | String | 收款人国籍 | O
| | bankCode(20) | String | 收款人账户银行编号 | O
| | bankAccountNumber(30) | String | 收款人银行账户 | O
| | bankAccountName(35) | String | 收款人账户银行名称 | O
| | bankAddress(35) | String | 收款人账户银行地址 | O

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientBeneficiaryNo(20) | String | 客户方收款人唯一编号
| | beneficiaryNo(20) | String | NextPls收款人唯一编号
msg | | String | 返回消息



## DoBeneficiaryDel
删除收款人 
### HTTP Request
<span class="http-method post">POST</span> `DO_BENEFICIARY_DEL`

> Request Body:

```json
{
    "apiName": "DO_BENEFICIARY_DEL",
    "entity": {
        "clientBeneficiaryNo": "TEST_B002",
        "beneficiaryNo": "XD201G0589941750"
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
             "clientBeneficiaryNo": "TEST_B002",
             "beneficiaryNo": "XD201G0589941750"
         }
     }'
```
```java
    public class example{
        public static void main(String[] args){
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsBeneficiaryRequestDto beneficiaryRequestDto = new NextPlsBeneficiaryRequestDto();
            beneficiaryRequestDto.setClientBeneficiaryNo("TEST_B002");
            beneficiaryRequestDto.setBeneficiaryNo("XD201G0589941750");
            NextPlsDoBeneficiaryDelRequest beneficiaryDelRequest = NextPlsDoBeneficiaryDelRequest.build(beneficiaryRequestDto);
            client.execute(beneficiaryDelRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientBeneficiaryNo(20) | String | 客户方收款人唯一编号 | M
| | BeneficiaryNo(20) | String | NextPls收款人唯一编号 | M

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_DEL_R",
    "code": "200",
    "entity": {},
    "msg": "success"
}
```

### Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
msg | | String | 返回消息

## GetRemitter
获取汇款人信息 
### HTTP Request
<span class="http-method post">POST</span> `GET_REMITTER`

> Request Body:

```json
{
    "apiName": "GET_REMITTER",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873"
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
             "clientRemitterNo": "TEST_B001",
             "remitterNo": "JJ201A1131599873"
         }
     }'
```
```java
    public class example{
        public static void main(String[] args){
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsRemitterRequestDto remitterRequestDto = new NextPlsRemitterRequestDto();
            remitterRequestDto.setClientRemitterNo("TEST_B001");
            NextPlsGetRemitterRequest remitterRequest = NextPlsGetRemitterRequest.build(remitterRequestDto);
            client.execute(remitterRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientRemitterNo(20) | String | 客户方汇款人唯一编号 | C
| | remitterNo(20) | String | NextPls汇款人唯一编号 | C

> Response Body:

```json
{
    "apiName": "GET_REMITTER_R",
    "code": "200",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873",
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
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientRemitterNo | String | 客户方汇款人唯一编号
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

## GetBeneficiary
获取指定收款人信息 
### HTTP Request
<span class="http-method post">POST</span> `GET_BENEFICIARY`

> Request Body:

```json
{
    "apiName": "GET_BENEFICIARY",
    "entity": {
        "clientBeneficiaryNo": "TEST_B002",
        "beneficiaryNo": "XD201G0589941750"
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
            "clientBeneficiaryNo": "TEST_B002",
            "beneficiaryNo": "XD201G0589941750"
        } 
     }'
```
```java
    public class example{
        public static void main(String[] args){
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsBeneficiaryRequestDto beneficiaryRequestDto = new NextPlsBeneficiaryRequestDto();
            beneficiaryRequestDto.setClientBeneficiaryNo("TESTB002");
            NextPlsGetBeneficiaryRequest beneficiaryRequest = NextPlsGetBeneficiaryRequest.build(beneficiaryRequestDto);
            client.execute(beneficiaryRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientBeneficiaryNo(20) | String | 客户方收款人唯一编号 | C
| | beneficiaryNo(20) | String | NextPls收款人唯一编号 | C

> Response Body:

```json
{
    "apiName": "GET_BENEFICIARY_R",
    "code": "200",
    "entity": {
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "XD201G0589941750",
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
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientBeneficiaryNo | String | 客户方收款人唯一编号
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


# 交易
## GetBalance
获取账户余额 
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
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
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
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsBalanceDto balanceDto = new NextPlsBalanceDto();
            balanceDto.setCurrency("HKD");
            NextPlsGetBalanceRequest balanceRequest = NextPlsGetBalanceRequest.build(balanceDto);
            client.execute(balanceRequest);             
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | currency(3) | String | 存款币种 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | partnerCode | String | 客户编号
| | currency | String | 币种
| | balance | String | 余额
msg | | String | 返回消息

## GetExRate
获取汇率 
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
             "payOutCurrency": "PHP"
         }
     }'
```
```java
    public class example{
        public static void main(String[] args){
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsExRateDto exRateDto = new NextPlsExRateDto();
            exRateDto.setPayInCurrency("HKD");
            exRateDto.setPayOutCurrency("PHP");
            NextPlsGetExRateRequest exRateRequest = NextPlsGetExRateRequest.build(exRateDto);
            client.execute(exRateRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | payInCurrency(3) | String | 存入币种 | M
| | payOutCurrency(3) | String | 到账币种 | M

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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | payInCurrency | String | 存入币种
| | payOutCurrency | String | 到账币种
| | exRate | String | 汇率
msg | | String | 返回消息

## DoTransactionPre
预创建订单 
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
curl -X POST https://open.remitly.com/partner/customer/create
    -H "Content-Type: application/json"
    -H ”Authorization:"your authorization"
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
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsTransactionPreRequestDto preRequestDto = new NextPlsTransactionPreRequestDto();
            preRequestDto.setClientTxnNo("1000");
            preRequestDto.setPayInCurrency("HKD");
            preRequestDto.setPayOutCurrency("PHP");
            preRequestDto.setTransferCurrency("HKD");
            preRequestDto.setTransferAmount("100");
            preRequestDto.setPaymentMode("Bank");
            NextPlsDoTransactionPreRequest preRequest = NextPlsDoTransactionPreRequest.build(preRequestDto);
            client.execute(preRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientTxnNo(20) | String | 客户方唯一编号 | M
| | payInCurrency(3) | String | 存入币种 | M
| | payOutCurrency(3) | String | 到账币种 | M
| | transferCurrency(3) | String | 结算币种 | M
| | transferAmount(18) | String | 结算金额 | M
| | paymentMode(20) | String | 支付方式 | M

> Response Body:

```json
{
    "apiName": "DO_TRANSACTION_PRE_R",
    "code": "200",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000",
        "payInCurrency": "HKD",
        "payoutCurrency": "PHP",
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
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | txnNo | String | 交易编号
| | clientTxnNo | String | 客户方交易编号
| | payInCurrency | String | 存入币种
| | payoutCurrency | String | 到账币种
| | payOutAmount | String | 到账金额
| | transferCurrency | String | 结算币种
| | transferAmount | String | 结算金额
| | paymentMode | String | 支付方式
| | exchangeRate | String | 汇率
| | commission | String | 手续费
| | totalAmount | String | 支付总额
msg | | String | 返回消息

## DoTransactionAdd
创建订单 
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
       "remitterNo": "JJ201A1131599873",
       "beneficiaryNo": "RP122141",
       "purposeCode": "3"
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
            
            NextPlsClient client = new DefaultNextPlsClient("http://staging.nextpls.com/v1/remittance", "test_client", "cek_tester_remit", "initial_tester01", publicKey, secretKey);
            NextPlsTransactionRequestDto txnRequestDto = new NextPlsTransactionRequestDto();
            txnRequestDto.setTxnNo("IU201G0279816077");
            txnRequestDto.setClientTxnNo("1000");
            txnRequestDto.setBeneficiaryNo("XD201G0589941750");
            txnRequestDto.setRemitterNo("JJ201A1131599873");
            txnRequestDto.setPurposeCode("3");
            txnRequestDto.setPayInCountry("HKG");
            txnRequestDto.setPayOutCountry("PHL");
            NextPlsDoTransactionRequest transactionAddRequest = NextPlsDoTransactionRequest.build(txnRequestDto);
            client.execute(transactionAddRequest);
          
        }
    }
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | TxnNo(20) | String | 订单编号 | M
| | clientTxnNo(20) | String | 客户方订单编号 | M
| | payInCountry(3) | String | 汇入币种 | M
| | payOutCountry(3) | String | 到账币种 | M
| | remitterNo(20) | String | 汇款人编号 | M
| | beneficiaryNo(20) | String | 收款人编号 | M
| | purposeCode(2) | String | 汇款目的编号 | M

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
参数 |  | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientTxnNo | String | 客户方订单唯一编号
| | txnNo | String | NextPls订单唯一编号
msg | | String | 返回消息

## GetTransactionStatus
获取交易状态
### HTTP Request
<span class="http-method post">POST</span> `GET_TRANSACTION_STATUS`

> Request Body:

```json
{
    "apiName": "GET_TRANSACTION_STATUS",
    "entity": {
        "clientTxnNo": "1000",
        "txnNo": "IU201G0279816077"
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
             "clientTxnNo": "1000",
             "txnNo": "IU201G0279816077"
         }
     }'
```

### Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | 调用接口名称 | M
entity | | Object | 客户方请求参数 | M
| | clientTxnNo(20) | String | 客户方订单唯一编号 | M
| | txnNo(20) | String | NextPls订单唯一编号 | M

> Response Body:

```json
{
    "apiName": "GET_TRANSACTION_STATUS_R",
    "code": "200",
    "entity": {
        "clientTxnNo": "1000",
        "txnNo": "IU201G0279816077",
        "status": "TRANSACTION_ING"
    },
    "msg": "success"
}
```

### Response Body
参数 |  | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | 被调用接口名称
code | | String | 返回码
entity | | Object | NextPls返回结果
| | clientTxnNo | String | 客户方订单唯一编号
| | txnNo | String | NextPls订单唯一编号
| | status | String | 订单状态
msg | | String | 返回消息

