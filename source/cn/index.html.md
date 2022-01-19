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

# 接口版本 v1.0.19.2 
[For English](/en)
## 简介

>我们建议首先使用 SDK 开始:

```json
  <dependency>
      <groupId>com.nextpls</groupId>
      <artifactId>sdk</artifactId>
      <version>1.0.19.2</version>
  </dependency>
```
```shell

  <dependency>
      <groupId>com.nextpls</groupId>
      <artifactId>sdk</artifactId>
      <version>1.0.19.2</version>
  </dependency>

```
```java
/**
  <dependency>
      <groupId>com.nextpls</groupId>
      <artifactId>sdk</artifactId>
      <version>1.0.19.2</version>
  </dependency>
*/
```

欢迎选择 NextPls! 我们会最大程度地保障您的资金和数据安全。

我们拥有一支优秀的开发团队。

如果想获取更多信息，可以访问我们的官方网站 [NextPls](https://www.nextpls.com) 更多地了解我们。

![avatar](/images/sequenceDiagram.jpg)

![avatar](/images/status.jpg)

# 开始
## 1.NextPls API的加密方式
### 1.1.Request

### 1.1.1.生成一个CEK

消息在发送到NextPls之前，所有的请求体都应该使用加密算法对请求体进行加密。因此，我们需要一个CEK(内容加密密钥),该密钥将被加密后放在请求头的Content-Code的字段中发送到NextPls服务器。
  
我们希望一个CEK由两个16位字符串拼接而成(16位的初始向量ivParameter和16位的AES密钥sKey)

`示例`

item | ASCII_string
--------- | -------
sKey | cek_tester_remit
ivParameter | initial_tester01
CEK | cek_tester_remitinitial_tester01
  
<aside class="notice">
CEK不需要有特殊含义，可以为随机的字符串
</aside>
  
### 1.1.2.利用CEK对方法体进行加密

将请求发送到NextPls之前，应使用CEK加密消息体部分

对消息体进行加密时，应使用AES/CBC/PKCS5Padding算法，之后再使用BASE64对结果进行转码。

>加密体加密示例:

```json
  {
    "Example":"Please move to java"
  }
```
```shell
    Please find the example in java
```
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
            System.out.println("The encrypted body:" + encryptedBody);
            
        }
    }
```

### 1.1.3.利用NextPls公钥对CEK进行加密

在调用API前，必须将加密后的CEK放在请求头的"Content-Code"参数中。应使用RSA算法对CEK进行加密，所有的客户都会获得由NextPls提供的一个用于CEK加密的专有公钥。

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

### 1.1.4.生成签名

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
  
### 1.1.5.生成请求头和请求体

使用加密的CEK、加密的方法体和签名便可组成一个完整的请求
  
`示例`

Header |
-------- |
Accept : application/base64 |
Authorization : AU1234567 |
Content-Code : dpcnD2YnJpCr0EIUNkz5so9rsBloBJRk/ie/awlgOV8ECoob3yB9QoAtP7LQNtOmziQOrFXueD0B62gc2f+AqCOs5D66sWQjlCWnjyqOkFjfow93CtYvGwpuoiqNArk39qvbe3lTo2g3qalg5YEe0Reohf2yHFbsYrwtgvBWQUI= |
Signature : CiELHoMm9ab9FduNLWQZ15OI4O+ms8xGjJBEMyRsLRvi9cQI0QctIajwnejCrri7q0ep2zf6Fqn9DIWpF2vgsAHK3SWr2SkykxY2Sheisv1hjpxOVYgnLDcI/0KXTFSjHQWQ+JG6d8VvwrAErqY0jk9pafUj1SfxrHpUzEUWDGQ= |

Body |
------ |
Deo7f9su8hdo0PCCKxyjuCRVCKAotP01jgfDJd82jrLQAvEyXK+hwNMF2mLKidCERaS604yzdQ2REQ0Rja/2H87VLLmsQx7Bkbe0yah8ALIaCabwY30aG/FPsjY4Y7OhujaEAzOVRUrV21iYDL5nUg= |

### 1.2.Response

### 1.2.1.验证签名

为了验证NextPls服务器的真实性，客户方需要将签名进行SHA256withRSA算法解密，并与加密请求体进行校验,如果结果不是true，这意味着无效的签名，响应主体的真实性得不到保证，客户不应该进行进一步的处理

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

### 1.2.2.CEK的获取

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

所有的响应体在从NextPls服务器发送之前都使用AES/CBC/PKCS5Padding算法进行加密。因此，它需要首先从http报头中的Content-Code字段中解析CEK(通过内容加密密钥)

CEK由32字节的随机字符(初始向量为16字节，AES密钥为16字节)组成。它是用客户提供的公钥(RSA)进行加密的。所以客户需要准备好相应的私钥。

CEK可以从响应头中的Content-Code字段中获得

### 1.2.3.用CEK解密响应体

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

## 2.1.Keys
在下面的文档中缩写值的定义

参数 | 类型 | 描述
--------- | ------- | -----------
M | string | 必填字段
O | string | 选填字段
C | string | 有前置条件的选填或必填字段

# 基础信息
## 3.1.GetCountryPayMode
获取指定国家支持的支付模式 
### 3.1.1.HTTP Request
<span class="http-method post">POST</span> `GET_COUNTRY_PAY_MODE`

接口暂未开放，敬请期待！

# 汇款人信息

## ~~4.1.DoRemitterAdd~~
添加汇款人信息（不开放）
### 4.1.1.HTTP Request
<span class="http-method post">POST</span> `DO_REMITTER_ADD`


## ~~4.2.DoRemitterEdit~~
修改汇款人信息
### 4.2.1.HTTP Request
<span class="http-method post">POST</span> `DO_REMITTER_EDIT`


## ~~4.3.GetRemitter~~
获取汇款人信息
### 4.3.1.HTTP Request
<span class="http-method post">POST</span> `GET_REMITTER`


# 收款人信息

## ~~5.1.DoBeneficiaryAdd~~
添加收款人信息
### 5.1.1.HTTP Request
<span class="http-method post">POST</span> `DO_BENEFICIARY_ADD`


## ~~5.2.DoBeneficiaryEdit~~
修改收款人信息
### 5.2.1.HTTP Request
<span class="http-method post">POST</span> `DO_BENEFICIARY_EDIT`


## ~~5.3.DoBeneficiaryDel~~
删除收款人 
### 5.3.1.HTTP Request
<span class="http-method post">POST</span> `DO_BENEFICIARY_DEL`


## ~~5.4.GetBeneficiary~~
获取指定收款人信息 
### 5.4.1.HTTP Request
<span class="http-method post">POST</span> `GET_BENEFICIARY`


# 交易
## 6.1.GetBalance
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
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

### 6.1.2.Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_BALANCE | M
entity | | Object | 客户方请求参数 | M
| | currency | String(3) | 存款币种 | M

> Response Body:

```json
{
  "apiName": "GET_BALANCE_R",
  "code": "200",
  "msg": "success",
  "entity": {
    "partnerCode": "test_client",
    "currency": "HKD",
    "balance": "1999.33"
  }
}
```

### 6.1.3.Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | GET_BALANCE_R
code | | String | 返回码
msg | | String | 返回消息
entity | | Object | NextPls返回结果
| | partnerCode | String | 客户编号
| | currency | String | 币种
| | balance | String | 余额

## 6.2.GetExRate
获取汇率 
### HTTP Request
<span class="http-method post">POST</span> `GET_EX_RATE`

```json
{
  "apiName": "GET_EX_RATE",
  "entity": {
    "payInCountry": "HKG",
    "payInCurrency": "HKD",
    "payOutCountry": "PHL",
    "payOutCurrency": "PHP"
  }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
    -d
    '{
         "apiName": "GET_EX_RATE",
         "entity": {
             "payInCountry": "HKG",
             "payInCurrency": "HKD",
             "payOutCountry": "PHL",
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
    exRateDto.setPayInCountry("HKG");
    exRateDto.setPayInCurrency("HKD");
    exRateDto.setPayOutCountry("PHL");
    exRateDto.setPayOutCurrency("PHP");
    NextPlsGetExRateRequest exRateRequest = NextPlsGetExRateRequest.build(exRateDto);
    client.execute(exRateRequest);

  }
}
```

### 6.2.2.Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_EX_RATE | M
entity | | Object | 客户方请求参数 | M
| | payInCountry | String(3) | 汇入国家 | M
| | payInCurrency | String(3) | 存入币种 | M
| | payOutCountry | String(3) | 到账国家 | M
| | payOutCurrency | String(3) | 到账币种 | M

> Response Body:

```json
{
  "apiName": "GET_REMITTER_R",
  "code": "200",
  "msg": "success",
  "entity": {
    "payInCountry": "HKG",
    "payInCurrency": "HKD",
    "payOutCountry": "PHL",
    "payOutCurrency": "PHP",
    "exRate": "7.369781"
  }
}
```

### 6.2.3.Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | GET_REMITTER_R
code | | String | 返回码
msg | | String | 返回消息
entity | | Object | NextPls返回结果
| | payInCountry | String | 存入国家
| | payInCurrency | String | 存入币种
| | payOutCountry | String | 到账国家
| | payOutCurrency | String | 到账币种
| | exRate | String | 汇率


## ~~6.3.GetExRateLock~~
This method allows the partner to Get the last rate and locked


## 6.4.DoTransactionPre
预创建订单 
### HTTP Request
<span class="http-method post">POST</span> `DO_TRANSACTION_PRE`

> Request Body:

```json
{
  "apiName": "DO_TRANSACTION_PRE",
  "entity": {
    "clientTxnNo": "1000",
    "transactionType": "C2C",
    "payInCountry": "HKG",
    "payInCurrency": "HKD",
    "payInAmount": "13.57",
    "payOutCountry": "PHL",
    "payOutCurrency": "PHP",
    "payOutAmount": "100",
    "transferCurrency": "PHP",
    "paymentMode": "Bank"
  }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
    -d
    '{
         "apiName": "DO_TRANSACTION_PRE",
         "entity": {
             "clientTxnNo": "1000",
             "transactionType": "C2C",
             "payInCountry": "HKG",
             "payInCurrency": "HKD",
             "payInAmount": "13.57",
             "payOutCountry": "PHL",
             "payOutCurrency": "PHP",
             "payOutAmount": "100",
             "transferCurrency": "PHP",
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
    preRequestDto.setTransactionType("C2C");
    preRequestDto.setPayInCountry("HKG");
    preRequestDto.setPayInCurrency("HKD");
    preRequestDto.setPayInAmount("13.57");
    preRequestDto.setPayOutCountry("PHL");
    preRequestDto.setPayOutCurrency("PHP");
    preRequestDto.setPayOutAmount("100");
    preRequestDto.setTransferCurrency("PHP");
    preRequestDto.setPaymentMode("Bank");
    NextPlsDoTransactionPreRequest preRequest =
      NextPlsDoTransactionPreRequest.build(preRequestDto);
    client.execute(preRequest);

  }
}
```

### 6.4.2.Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_TRANSACTION_PRE | M
entity | | Object | 客户方请求参数 | M
| | clientTxnNo | String(20) | 客户方唯一编号 | M
| | transactionType | String(3) | 交易类型：C2C(默认),C2B,B2B,B2C | O
| | payInCountry | String(3) | 汇款(源)国家 | M
| | payInCurrency | String(3) | 汇款(源)币种 | M
| | payInAmount | String(18) | 汇款(源)金额 | C
| | payOutCountry | String(3) | 到账(目标)国家 | M
| | payOutCurrency | String(3) | 到账(目标)币种 | M
| | payOutAmount | String(18) | 到账(目标)金额 | C
| | transferCurrency | String(3) | 交易币种(固定侧, 必须是汇款或到账币种) | M
| | paymentMode | String(20) | 支付方式 | M

> Response Body:

```json
{
  "apiName": "DO_TRANSACTION_PRE_R",
  "code": "200",
  "msg": "success",
  "entity": {
    "txnNo": "IU201G0279816077",
    "clientTxnNo": "1000",
    "transactionType": "C2C",
    "payInAmount": "13.57",
    "payInCurrency": "HKD",
    "payOutCurrency": "PHP",
    "payOutAmount": "100",
    "transferCurrency": "HKD",
    "paymentMode": "Bank",
    "exchangeRate": "7.369781",
    "commission": "0.1000",
    "commissionCurrency": "HKD",
    "totalAmount": "13.67"
  }
}
```

### 6.4.3.Response Body
参数 |   | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_PRE_R
code | | String | 返回码
msg | | String | 返回消息
entity | | Object | NextPls返回结果
| | txnNo | String | 交易编号
| | clientTxnNo | String | 客户方交易编号
| | transactionType | String | 交易类型
| | payInAmount | String | 汇款(源)金额
| | payInCurrency | String | 汇款(源)币种
| | payOutCurrency | String | 到账(目标)币种
| | payOutAmount | String | 到账(目标)金额
| | transferCurrency | String | 交易币种
| | paymentMode | String | 支付方式
| | exchangeRate | String | 汇率
| | commission | String | 手续费
| | commissionCurrency | String | 手续费币种
| | totalAmount | String | 支付总额

## 6.5.DoTransaction
创建订单 
### 6.5.1.HTTP Request
<span class="http-method post">POST</span> `DO_TRANSACTION`

> Request Body:

```json
{
    "apiName": "DO_TRANSACTION",
    "entity": {
       "txnNo": "IU201G0279816077",
       "clientTxnNo": "1000",
       "remitterNo": "",
       "beneficiaryNo": "",
       "purposeCode": "EDUCATION",
       
        "remitterFirstName": "Remitter_First_Name",
        "remitterMiddleName": "",
        "remitterLastName": "Remitter_Last_Name",
        "remitterFirstLocalName": "Remitter_First_Local_Name",
        "remitterMiddleLocalName": "",
        "remitterLastLocalName": "Remitter_Middle_Last_Name",
        "remitterMobile": "12345678910",
        "remitterGender": "M",
        "remitterBirthdate": "01/01/1994",
        "remitterEmail": "nextPls@nextPls.com",
        "remitterAddress1": "Italy",
        "remitterAddress2": "",
        "remitterAddress3": "",
        "remitterPostalCode": "",
        "remitterOccupation": "",
        "remitterIdType": "PASSPORT",
        "remitterIdNumber": "PS256454165",
        "remitterIdDesc": "",
        "remitterIdIssueDate": "01/01/1994",
        "remitterIdExpDate": "01/01/1994",
        "remitterNationality": "HKG",
        "remitterAccountNumber": "",
        "sourceOfIncome": "SALARY",
        
        "beneficiaryFirstName": "Beneficiary_First_Name",
        "beneficiaryMiddleName": "",
        "beneficiaryLastName": "Beneficiary_Last_Name",
        "beneficiaryFirstLocalName": "Beneficiary_First_Local_Name",
        "beneficiaryMiddleLocalName": "",
        "beneficiaryLastLocalName": "Beneficiary_Last_Local_Name",
        "beneficiaryMobile": "12345678910",
        "beneficiaryGender": "",
        "beneficiaryBirthdate": "",
        "beneficiaryEmail": "",
        "beneficiaryAddress1": "Philippines",
        "beneficiaryAddress2": "",
        "beneficiaryAddress3": "",
        "beneficiaryPostalCode": "",
        "beneficiaryOccupation": "",
        "beneficiaryIdType": "PASSPORT",
        "beneficiaryIdNumber": "",
        "beneficiaryIdDesc": "",
        "beneficiaryIdIssueDate": "",
        "beneficiaryIdExpDate": "",
        "beneficiaryNationality": "HKG",
        "beneficiaryRelationship": "BROTHER",
        "beneficiaryBankCode": "11003544",
        "beneficiaryBankAccountNumber": "4555556564564",
        "beneficiaryBankAccountName": "Benificiary_BankAccountName",
        "beneficiaryAccount": "",
        "beneficiaryBankAddress": ""
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
    -d
    '{
         "apiName": "DO_TRANSACTION",
         "entity": {
                       "txnNo": "IU201G0279816077",
                       "clientTxnNo": "1000",
                       "remitterNo": "JJ201A1131599873",
                       "beneficiaryNo": "RP122141",
                       "purposeCode": "EDUCATION"
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
    txnRequestDto.setPurposeCode("EDUCATION");
    NextPlsDoTransactionRequest transactionAddRequest =
      NextPlsDoTransactionRequest.build(txnRequestDto);
    client.execute(transactionAddRequest);

  }
}
```

### 6.5.2.Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_TRANSACTION | M
entity | | Object | 客户方请求参数 | M
| | txnNo | String(20) | 订单编号 | M
| | clientTxnNo | String(20) | 客户方订单编号 | M
| | purposeCode | String(16) | 汇款目的编号 | M
| | remitterFirstName | String(50) | 汇款人名-英 | M
| | remitterMiddleName | String(50) | 汇款人中间名-英 | O
| | remitterLastName | String(50) | 汇款人姓-英 | M
| | remitterFirstLocalName | String(50) | 汇款人名-母语 | M
| | remitterMiddleLocalName | String(50) | 汇款人中间名-母语 | O
| | remitterLastLocalName | String(50) | 汇款人姓-母语 | M
| | remitterMobile | String(20) | 汇款人手机号 | M
| | remitterEmail | String(50) | 汇款人邮箱 | O
| | remitterAddress1 | String(35) | 汇款人地址1 | M
| | remitterAddress2 | String(35) | 汇款人地址2 | O
| | remitterAddress3 | String(50) | 汇款人地址3 | O
| | remitterPostalCode | String(16) | 汇款人邮编 | C
| | remitterOccupation | String(64) | 汇款人职业 | C
| | remitterIdType | String(16) | 汇款人证件类型 | M
| | remitterIdNumber | String(20) | 汇款人证件号 | M
| | remitterIdDesc | String(30) | 汇款人证件描述 | C
| | remitterIdIssueDate | String(10) | 汇款人证件签发日期 (MM/DD/YYYY) | O
| | remitterIdExpDate | String(10) | 汇款人证件有效期 (MM/DD/YYYY)  | O
| | remitterBirthdate | String(10) | 汇款人生日 (MM/DD/YYYY) | O
| | remitterCountryOfBirth | String(10) | 汇款人出生国家 | O
| | remitterGender | String(1) | 汇款人性别. M=Male, F=Female | O
| | remitterNationality | String(3) | 汇款人国籍(3位国家编码) | M
| | remitterAccountNumber | String(30) | 汇款人账户号 | O
| | sourceOfIncome | String(16) | 汇款人收入来源 | M
| | beneficiaryFirstName | String(50) | 收款人名-英 | M
| | beneficiaryMiddleName | String(50) | 收款人中间名-英 | O
| | beneficiaryLastName | String(50) | 收款人姓-英 | M
| | beneficiaryFirstLocalName | String(50) | 收款人名-母语 | M
| | beneficiaryMiddleLocalName | String(50) | 收款人中间名-母语 | O
| | beneficiaryLastLocalName | String(50) | 收款人姓-母语 | M
| | beneficiaryMobile | String(20) | 收款人手机号 | M
| | beneficiaryEmail | String(50) | 收款人邮箱 | O
| | beneficiaryAddress1 | String(35) | 收款人地址1 | M
| | beneficiaryAddress2 | String(35) | 收款人地址2 | O
| | beneficiaryAddress3 | String(35) | 收款人地址3 | O
| | beneficiaryPostalCode | String(16) | 收款人邮编 | C
| | beneficiaryOccupation | String(64) | 收款人职业 | C
| | beneficiaryIdType | String(16) | 收款人证件类型 | O
| | beneficiaryIdNumber | String(20) | 收款人证件号码 | O
| | beneficiaryIdDesc | String(20) | 收款人证件描述 | C
| | beneficiaryIdIssueDate | String(10) | 证件签发日期(MM/DD/YYYY) | O
| | beneficiaryIdExpDate | String(10) | 证件有效日期(MM/DD/YYYY) | O
| | beneficiaryBirthdate | String(10) | 收款人生日(MM/DD/YYYY) | O
| | beneficiaryCountryOfBirth | String(3) | 收款人出生国家 | O
| | beneficiaryGender | String(1) | 收款人性别 | O
| | beneficiaryNationality | String(3) | 收款人国籍(3位国家编码) | M
| | beneficiaryRelationship | String(16) | 与汇款者的关系 | M
| | beneficiaryBankCode | String(20) | 收款机构编号 | C
| | beneficiaryBankName | String(64) | 收款银行名称 | C
| | beneficiaryBankAccountNumber | String(30) | 收款银行账户 | C
| | beneficiaryBankAccountName | String(35) | 收款户名 | C
| | beneficiaryAccount | String(35) | 收款账号 | C
| | beneficiaryAccountType | String(35) | 收款账号类型 | C
| | beneficiaryBankAddress | String(35) | 收款银行地址 | O
| | beneficiaryIban | String(35) | 收款人银行IBAN | C
| | beneficiarySwiftCode | String(16) | 收款人银行Swift Code | C
| | beneficiaryRoutingCode | String(16) | 收款人银行Routing Code | C

> Response Body:

```json
{
  "apiName": "DO_TRANSACTION_R",
  "code": "200",
  "msg": "success",
  "entity": {
    "txnNo": "IU201G0279816077",
    "clientTxnNo": "1000",
    "status": "TRANSACTION_ING"
  }
}
```

### 6.5.3.Response Body
参数 |  | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_R
code | | String | 返回码
msg | | String | 返回消息
entity | | Object | NextPls返回结果
| | txnNo | String | NextPls订单唯一编号
| | clientTxnNo | String | 客户方订单唯一编号
| | status | String | 订单状态
| | errorMsg | String | 订单错误信息


## ~~6.6.DoTokenTransaction~~

## 6.7.GetTransactionStatus
获取交易状态
### 6.7.1.HTTP Request
<span class="http-method post">POST</span> `GET_TRANSACTION_STATUS`

> Request Body:

```json
{
    "apiName": "GET_TRANSACTION_STATUS",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000"
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
    -d
    '{
         "apiName": "GET_TRANSACTION_STATUS",
         "entity": {
             "txnNo": "IU201G0279816077",
             "clientTxnNo": "1000"
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
        NextPlsGetTxnStatusRequest txnStatusRequest = 
                    NextPlsGetTxnStatusRequest.build(txnRequestDto);
        client.execute(txnStatusRequest);
      
    }
}
```

### 6.7.2.Request Body
参数 |  | 类型 | 描述 | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_TRANSACTION_STATUS | M
entity | | Object | 客户方请求参数 | M
| | txnNo | String(20) | NextPls订单唯一编号 | M
| | clientTxnNo | String(20) | 客户方订单唯一编号 | M

> Response Body:

```json
{
  "apiName": "GET_TRANSACTION_STATUS_R",
  "code": "200",
  "msg": "success",
  "entity": {
    "reference": "FRTR1001349991",
    "txnNo": "IU201G0279816077",
    "clientTxnNo": "1000",
    "status": "TRANSACTION_ING"
  }
}
```

### Response Body
参数 |  | 类型 | 描述
--------- | ------- | ------- |-----------
apiName | | String | GET_TRANSACTION_STATUS_R
code | | String | 返回码
msg | | String | 返回消息
entity | | Object | NextPls返回结果
| | txnNo | String | NextPls订单唯一编号
| | clientTxnNo | String | 客户方订单唯一编号
| | reference | String | 现金到账的领取码
| | status | String | 订单状态
| | errorMsg | String | 订单错误信息


## ~~6.8.DoTransactionCancel~~


# 错误信息
## 7.1.HTTP Error Codes
### 这些错误编码将在API的返回体中显示

<aside class="warning">
注意：接口状态仅代表当前请求结果，不代表订单状态，订单状态请务必以订单查询中的状态为准！
</aside>

Error Code | Description
--------- | -------
200 | Request Success
500 | Request Fail
10000 | The system is busy, please try again later
10006 | No this Method
10007 | Invalid Signature
10010 | Missing Parameters
10011 | Wrong Parameters
10012 | Interface is not open
|
10101 | Fee amount query error
10102 | The rate query failed
10103 | Please get the rate first
10104 | The target country error or unsupported
10105 | The source country error or unsupported
10106 | The target currency error or unsupported
10107 | The source currency error or unsupported
|
20010 | The remitter add failed
20011 | The clientRemitterNo already exists
20013 | The remitter query failed
20014 | The remitter update failed
20050 | The beneficiary add failed
20051 | The clientBeneficiaryNo already exists
21052 | The beneficiary detail doesn't exist
20053 | The beneficiary query failed
20054 | The beneficiary update failed
|
30001 | The clientTxnNo already exists
30002 | The transaction query failed
30003 | The transaction create failed
30004 | The transaction amount is empty
30005 | The amount received and remitted does not match
30006 | The transaction had time out
30007 | The transaction has been cancelled
30008 | The transaction cancellation failed
30009 | The transaction doesn't exist
30010 | The transaction status query failed
30011 | The token of rate doesn't exist or time out
30012 | The target amount error
30013 | This payment mode unsupported
30014 | Suspected duplicate transaction
30015 | It is not supported that source and settlement currency are different
|
50000 | High risk interception
50011 | The amount exceeds the single limit
50012 | The amount exceeds the daily limit
50013 | The amount exceeds the monthly limit
|
88001 | No account opened in this currency
88002 | Insufficient balance of account/credit
