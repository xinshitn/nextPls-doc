---
title: API NextPls

language_tabs: # must be one of https://git.io/vQNgJ
  - json
  - shell
  - java

toc_footers:
  - <a href='#'>Contact Us</a>
  - <a href='https://www.nextpls.com'>Powered by NextPls</a>

includes:

search: true
---

# Version 1.0.12

## Introduction

>We suggest using SDK to start first (Example in Java):

```json
  <dependency>
      <groupId>com.nextpls</groupId>
      <artifactId>sdk</artifactId>
      <version>1.0.12</version>
  </dependency>
```
```shell

  <dependency>
      <groupId>com.nextpls</groupId>
      <artifactId>sdk</artifactId>
      <version>1.0.12</version>
  </dependency>

```
```java
/**
  <dependency>
      <groupId>com.nextpls</groupId>
      <artifactId>sdk</artifactId>
      <version>1.0.12</version>
  </dependency>
*/
```

Welcome the API document for NextPls!

Fueled by a fundamental belief that having access to financial services creates opportunity,
Nextpls is committed to democratizing financial services and empowering people and businesses 
to join and thrive in the global economy. Since 2017 we've been driven by a simple goal - 
to build a digital network connecting people and business in all corners of the globe,
always striving to create opportunities for our clients. No exceptions.


# Getting Started
## 1.Cryptography in NextPls API 
### 1.1.Request

### 1.1.1.Generating a CEK

All request body should be encrypted with AES128 algorithm before sending to the NextPls server. So it needs a CEK(Content Encryption Key) which will be delivered to NextPls server as described in the Content-Code field in the http header.
  
Agent who wants to make a request API should generate an AES-128 key for the CEK, which is comprised of 32 bytes random digits (16 bytes for initial vectors and 16 bytes for AES key). 
  
`Example`

item | ASCII_string 
--------- | -------
sKey | cek_tester_remit
ivParameter | initial_tester01
CEK | cek_tester_remitinitial_tester01

<aside class="notice">
CEK is not a necessarily human readable ASCII string
</aside>
  
### 1.1.2.Encrypting body part with the CEK

>An example of encrypting the body (Example in Java):

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

Agent who wants to make a request API should encrypt the body part with CEK before sending to the NextPls server

When encrypt, you should use the algorithm of "AES/CBC/PKCS5Padding", then encoded of Base64.

### 1.1.3.Encrypting CEK with NextPls public key

>Example for CEK：

```java
    public class example{
        public static void main(String[] args){
            
            Cipher cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
            byte[] sbt = source.getBytes(ENCODING);
            byte[] epByte = cipher.doFinal(sbt);
            String encryptedCEK = Base64.getEncoder().encodeToString(epByte);
            System.out.println("The CEK(Content-Code) what be encrypted of RSA:" + encryptedCEK);
            
        }
    }
```

Agent who wants to make request API call MUST attach the CEK in the http header("Content-Code") as encrypted one. 

Encryption algorithm used for CEK protection is RSA-1024. Finally, the result needs to be encoded of Base64. 

All Agent will receive a public key from NextPls, which is used for encryption of the CEK.
  
### 1.1.4.Generating signature

>Example for Signature：

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

Agent who wants to make a request API call MUST attach a signature value of the encrypted body(The result in step 2) in the http header("Signature"). 

Signature is used for non-repudiation of the request body from an Agent. 

Generate signed value with sender’s private key.

Signature algorithm is Sha256WithRSA, And then also needs to be encoded of Base64. 
  
### 1.1.5.Generating request header and body

With encrypted CEK(step.3), encrypted body(step.2) and HMAC(step.4) values, the Authorization is partner code. Agent can generate a http header like followings;  
  
`Example`

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

### 1.2.1.Verifying signature

>Example for Verifying signature：

```java
    public class example{
        public static void main(String[] args){
          
            Signature signetcheck = Signature.getInstance(SIGNATURE_ALGORITHM);
            signetcheck.initVerify(getPublicKey());
            signetcheck.update(data.getBytes(ENCODING));
            boolean result = signetcheck.verify(Base64.getDecoder().decode(sign));
            System.out.println("Verifying signature：" + result);
            
        }
    }
```

To verify authenticity of NextPls server, Agent should calculate verification value from the encrypted body and compare with what received signed value described in the http header.

### 1.2.2.Deriving CEK

>Example for Deriving CEK

```java
    public class example{
        public static void main(String[] args){
            
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
            System.out.println("CEK is：" + CEK);
          
        }
    }
```

All response body is encrypted with AES128-CBC algorithm before sending from the NextPls server. So it needs to derive the CEK(Content Encryption Key) first from the Content-Code field in the http header. 

CEK is comprised of 32 bytes random digits (16 bytes for initial vectors and 16 bytes for AES key). It is encrypted with Agent’s public key (RSA-1024). So Agent needs to prepare its corresponding private key. 

CEK can be derived from ‘Content-Code’ field in response header. 

### 1.2.3.Decrypting body part with CEK

>Example for decrypting body part with CEK

```java
    public class example{
        public static void main(String[] args){
            
            byte[] raw = sKey.getBytes("ASCII");
            SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            IvParameterSpec iv = new IvParameterSpec(ivParameter.getBytes());
            cipher.init(Cipher.DECRYPT_MODE, skeySpec, iv);

            byte[] encrypted1 = Base64.getDecoder().decode(sSrc);
            byte[] original = cipher.doFinal(encrypted1);
            String encodingFormat = "UTF-8";
            String resBody = new String(original, encodingFormat);
            System.out.println("The result is：" + resBody);
          
        }
    }
```

Agent can extract plain JSON body with the CEK derived at above.

## 2.1.Keys

Definition of abbreviated values found in the documentation.

Parameter | Type | Description
--------- | ------- | -----------
M | string | value for this field is required
O | string | value for this field is optional
C | string | The requirement for this field is conditional based on other field values

# Basic Information
## 3.1.GetCountryPayMode
This method allows the partner to Get Country based Payment Modes.
### HTTP Request
<span class="http-method post">POST</span> `GET_COUNTRY_PAY_MODE`

The function is not open yet, Coming soon!

# Remitter

## 4.1.DoRemitterAdd
This method allows the partner to Registered Remitter Profile. 
### 4.1.1.HTTP Request
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
        "gender": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Italy",
        "address2": "",
        "address3": "",
        "postalCode": "",
        "occupation": "",
        "idType": "PASSPORT",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "accountNumber": "",
        "sourceIncome": "SALARY"
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
         "apiName": "DO_REMITTER_ADD",
         "entity": {
             "clientRemitterNo": "TEST_B001",
             "firstName": "Remitter_First_Name",
             "middleName": "Remitter_Middle_Name",
             "lastName": "Remitter_Last_Name",
             "mobile": "12345678910",
             "gender": "M",
             "birthdate": "01/01/1994",
             "email": "nextPls@nextPls.com",
             "address1": "Italy",
             "address2": "",
             "address3": "",
             "postalCode": "",
             "occupation": "",
             "idType": "PASSPORT",
             "idNumber": "PS256454165",
             "idDesc": "",
             "idIssueDate": "01/01/1994",
             "idExpDate": "01/01/1994",
             "nationality": "HKG",
             "accountNumber": "",
             "sourceIncome": "SALARY"
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
        NextPlsRemitterRequestDto remitterRequestDto = new NextPlsRemitterRequestDto();
        remitterRequestDto.setClientRemitterNo("TEST_R001");
        // ...
        NextPlsDoRemitterAddRequest remitterAddRequest = 
                NextPlsDoRemitterAddRequest.build(remitterRequestDto);
        client.execute(remitterAddRequest);
      
    }
}
```

### 4.1.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_REMITTER_ADD | M
entity | | Object | Parameter list | M
| | clientRemitterNo | String(20) | Unique code for partner remitter | M
| | firstName | String(50) | Remitter first name | M
| | middleName | String(50) | Remitter middle name | O
| | lastName | String(50) | Remitter last name | M
| | mobile | String(20) | Remitter mobile number | M
| | email | String(50) | The email id of remitter | O
| | address1 | String(35) | Remitter Address1 | M
| | address2 | String(35) | Remitter Address2 | O
| | address3 | String(50) | Remitter Address3 | O
| | postalCode | String(16) | Remitter PostalCode | C
| | occupation | String(64) | Remitter Occupation | C
| | idType | String(16) | Remitter identity type | M
| | idNumber | String(20) | ID number | M
| | idDesc | String(30) | description for id | C
| | idIssueDate | String(10) | ID issue date (MM/DD/YYYY) | O
| | idExpDate | String(10) | ID expiry date (MM/DD/YYYY)  | O
| | birthdate | String(10) | Remitter date of birth (MM/DD/YYYY) | O
| | gender | String(1) | Remitter gender. M=Male, F=Female | O
| | nationality | String(3) | Remitter Nationality(3 characters Country ISO code) | M
| | accountNumber | String(30) | Remitter account number | O
| | sourceIncome | String(16) | Remitter source of income | M

> Response Body:

```json
{
    "apiName": "DO_REMITTER_ADD_R",
    "code": "200",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873"
    },
    "msg": "success"
}
```

### 4.1.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_REMITTER_ADD_R
code | | String | Result Code
entity | | Object | Parameter list
| | clientRemitterNo | String | Unique code for partner remitter
| | remitterNo | String | Unique code for NextPls remitter
msg | | String | Result message

## 4.2.DoRemitterEdit
This method allows the partner to Edit Registered Remitter Profile.
### 4.2.1.HTTP Request
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
        "gender": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "postalCode": "",
        "occupation": "",
        "idType": "PASSPORT",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "accountNumber": "",
        "sourceIncome": "SALARY"
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
         "apiName": "DO_REMITTER_EDIT",
         "entity": {
             "clientRemitterNo": "TEST_B001",
             "remitterNo": "JJ201A1131599873",
             "firstName": "Remitter_First_Name",
             "middleName": "Remitter_Middle_Name",
             "lastName": "Remitter_Last_Name",
             "mobile": "12345678910",
             "gender": "M",
             "birthdate": "01/01/1994",
             "email": "nextPls@nextPls.com",
             "address1": "Philippines",
             "address2": "",
             "address3": "",
             "postalCode": "",
             "occupation": "",
             "idType": "PASSPORT",
             "idNumber": "PS256454165",
             "idDesc": "",
             "idIssueDate": "01/01/1994",
             "idExpDate": "01/01/1994",
             "nationality": "HKG",
             "accountNumber": "",
             "sourceIncome": "SALARY"
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
        NextPlsRemitterRequestDto remitterRequestDto = new NextPlsRemitterRequestDto();
        remitterRequestDto.setClientRemitterNo("TEST_R001");
        // ...
        NextPlsDoRemitterEditRequest remitterEditRequest = 
                NextPlsDoRemitterEditRequest.build(remitterRequestDto);
        client.execute(remitterEditRequest);
      
    }
}
```

### 4.2.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_REMITTER_EDIT | M
entity | | Object | Parameter list | M
| | clientRemitterNo | String(20) | Remitter number of partner | M
| | remitterNo | String(20) | Remitter number of NextPls | M
| | firstName | String(50) | Remitter first name | O
| | middleName | String(50) | Remitter middle name | O
| | lastName | String(50) | Remitter last name | O
| | mobile | String(20) | Remitter mobile number | O
| | email | String(50) | The email id of remitter | O
| | address1 | String(35) | Remitter Address1 | O
| | address2 | String(35) | Remitter Address2 | O
| | address3 | String(50) | Remitter Address3 | O
| | postalCode | String(16) | Remitter PostalCode | C
| | occupation | String(64) | Remitter Occupation | C
| | idType | String(16) | Remitter identity type | O
| | idNumber | String(20) | ID number | O
| | idDesc | String(30) | description for id | C
| | idIssueDate | String(10) | ID issue date (MM/DD/YYYY) | O
| | idExpDate | String(10) | ID expiry date (MM/DD/YYYY)  | O
| | birthdate | String(10) | Remitter date of birth (MM/DD/YYYY) | O
| | gender | String(1) | Remitter gender. M=Male, F=Female | O
| | nationality | String(3) | Remitter Nationality(3 characters Country ISO code) | O
| | accountNumber | String(30) | Remitter account number | O
| | sourceIncome | String(16) | Remitter source of income | O

> Response Body:

```json
{
    "apiName": "DO_REMITTER_EDIT_R",
    "code": "200",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873"
    },
    "msg": "success"
}
```

### 4.2.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_REMITTER_EDIT_R
code | | String | Result Code
entity | | Object | Parameter list
| | clientRemitterNo | String | Unique code for partner remitter
| | remitterNo | String | Unique code for NextPls remitter
msg | | String | Result message


## 4.3.GetRemitter
This method allows the partner to Get Registered Remitter Profile.  
### 4.3.1.HTTP Request
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
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
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsRemitterRequestDto remitterRequestDto = new NextPlsRemitterRequestDto();
        remitterRequestDto.setClientRemitterNo("TEST_B001");
        NextPlsGetRemitterRequest remitterRequest = 
                NextPlsGetRemitterRequest.build(remitterRequestDto);
        client.execute(remitterRequest);
      
    }
}
```

### 4.3.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_REMITTER | M
entity | | Object | Parameter list | M
| | clientRemitterNo | String(20) | Unique code for partner remitter | C
| | remitterNo | String(20) | Unique code for NextPls remitter | C

> Response Body:

```json
{
    "apiName": "GET_REMITTER_R",
    "code": "200",
    "msg": "success",
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873",
        "firstName": "REMITTER_First_Name",
        "middleName": "REMITTER_Middle_Name",
        "lastName": "REMITTER_Last_Name",
        "mobile": "12345678910",
        "gender": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "postalCode": "",
        "occupation": "",
        "idType": "PASSPORT",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "accountNumber": "",
        "sourceIncome": "SALARY"
    }
}
```

### 4.3.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_REMITTER_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | clientRemitterNo | String | Unique code for partner remitter 
| | remitterNo | String | Unique code for NextPls remitter
| | firstName | String | Remitter first name
| | middleName | String | Remitter middle name
| | lastName | String | Remitter last name
| | mobile | String | Remitter mobile number
| | email | String | The email id of remitter
| | address1 | String | Remitter Address1
| | address2 | String | Remitter Address2
| | address3 | String | Remitter Address3
| | postalCode | String | Remitter PostalCode
| | occupation | String | Remitter Occupation
| | idType | String | Remitter identity type
| | idNumber | String | ID number
| | idDesc | String | description
| | idIssueDate | String | ID issue date (MM/DD/YYYY)
| | idExpDate | String | ID expiry date (MM/DD/YYYY)
| | birthdate | String | Remitter date of birth (MM/DD/YYYY)
| | gender | String | Remitter gender
| | nationality | String | Remitter Nationality
| | accountNumber | String | Remitter account number
| | sourceIncome | String | Remitter source of income



# Beneficiary

## 5.1.DoBeneficiaryAdd
This method allows the partner to Register New Beneficiary.
### 5.1.1.HTTP Request
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
        "postalCode": "",
        "occupation": "",
        "idType": "PASSPORT",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "birthdate": "01/01/1994",
        "gender": "M",
        "nationality": "HKG",
        "relationship": "BROTHER",
        "bankCode": "11003544",
        "bankAccountNumber": "4555556564564",
        "bankAccountName": "Beneficiary_BankName",
        "account": "",
        "bankAddress": "Beneficiary_BankAddress"
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
                 "postalCode": "",
                 "occupation": "",
                 "idType": "PASSPORT",
                 "idNumber": "PS256454165",
                 "idDesc": "",
                 "idIssueDate": "01/01/1994",
                 "idExpDate": "01/01/1994",
                 "birthdate": "01/01/1994",
                 "gender": "M",
                 "nationality": "HKG",
                 "relationship": "BROTHER",
                 "bankCode": "11003544",
                 "bankAccountNumber": "4555556564564",
                 "bankAccountName": "Beneficiary_BankName",
                 "account": "",
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

### 5.1.2.Request Body
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
| | postalCode | String(16) | Beneficiary Postal Code | C
| | occupation | String(64) | Beneficiary Occupation | C
| | idType | String(16) | Type of Beneficiary Id Proof | O
| | idNumber | String(20) | Beneficiary ID Number | O
| | idDesc | String(20) | Description of Beneficiary ID | C
| | idIssueDate | String(10) | Issue date(MM/DD/YYYY) | O
| | idExpDate | String(10) | Expiry date(MM/DD/YYYY) | O
| | birthdate | String(10) | Beneficiary BirthDate(MM/DD/YYYY) | O
| | gender | String(1) | Gender of Beneficiary | O
| | nationality | String(3) | Nationality of Beneficiary(3 Character Country ISO Code) | M
| | relationship | String(16) | Relationship with the remitter | M
| | bankCode | String(20) | Bank code for Beneficiary | C
| | bankAccountNumber | String(30) | Bank Account Number of Beneficiary | C
| | bankAccountName | String(35) | Bank Account name of Beneficiary | C
| | account | String(35) | Account of Beneficiary | C
| | bankAddress | String(35) | Beneficiary Bank Address | O

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_ADD_R",
    "code": "200",
    "msg": "success",
    "entity": {
       "clientBeneficiaryNo": "BE1231211",
       "beneficiaryNo": "XD201G0589941750"
    }
}
```

### 5.1.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_BENEFICIARY_ADD_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | clientBeneficiaryNo | String | Unique code for partner beneficiary
| | beneficiaryNo | String | Unique code for NextPls beneficiary


## 5.2.DoBeneficiaryEdit
This method allows the partner to Edit Registered Beneficiary Profile.
### 5.2.1.HTTP Request
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
        "postalCode": "",
        "occupation": "",
        "idType": "PASSPORT",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "birthdate": "01/01/1994",
        "gender": "M",
        "nationality": "HKG",
        "relationship": "BROTHER",
        "bankCode": "11003544",
        "bankAccountNumber": "4555556564564",
        "bankAccountName": "Beneficiary_BankName",
        "account": "",
        "bankAddress": "Beneficiary_BankAddress"
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
             "postalCode": "",
             "occupation": "",
             "idType": "PASSPORT",
             "idNumber": "PS256454165",
             "idDesc": "",
             "idIssueDate": "01/01/1994",
             "idExpDate": "01/01/1994",
             "birthdate": "01/01/1994",
             "gender": "M",
             "nationality": "HKG",
             "relationship": "BROTHER",
             "bankCode": "11003544",
             "bankAccountNumber": "4555556564564",
             "bankAccountName": "Beneficiary_BankName",
             "account": "",
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

### 5.2.2.Request Body
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
| | postalCode | String(16) | Beneficiary Postal Code | C
| | occupation | String(64) | Beneficiary Occupation | C
| | idType | String(16) | Type of Beneficiary Id Proof | O
| | idNumber | String(20) | Beneficiary ID Number | O
| | idDesc | String(20) | Description of Beneficiary ID | O
| | idIssueDate | String(10) | Issue date(MM/DD/YYYY) | O
| | idExpDate | String(10) | Expiry date(MM/DD/YYYY) | O
| | birthdate | String(10) | Beneficiary BirthDate(MM/DD/YYYY) | O
| | gender | String(1) | Gender of Beneficiary | O
| | nationality | String(3) | Nationality of Beneficiary(3 Character Country ISO Code) | O
| | relationship | String(16) | Relationship with the remitter | M
| | bankCode | String(20) | Bank code for Beneficiary | O
| | bankAccountNumber | String(30) | Bank Account Number of Beneficiary | O
| | bankAccountName | String(35) | Bank Account name of Beneficiary | O
| | account | String(35) | Account of Beneficiary | C
| | bankAddress | String(35) | Beneficiary Bank Address | O

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_EDIT_R",
    "code": "200",
    "msg": "success",
    "entity": {
       "clientBeneficiaryNo": "TEST_B001",
       "beneficiaryNo": "XD201G0589941750"
    }
}
```

### 5.2.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_BENEFICIARY_EDIT_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | clientBeneficiaryNo | String | Unique code for partner beneficiary
| | beneficiaryNo | String | Unique code for NextPls beneficiary


## 5.3.DoBeneficiaryDel
This method allows the partner to Delete Registered beneficiary.
### 5.3.1.HTTP Request
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
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
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsBeneficiaryRequestDto beneficiaryRequestDto = new NextPlsBeneficiaryRequestDto();
        beneficiaryRequestDto.setClientBeneficiaryNo("TEST_B002");
        beneficiaryRequestDto.setBeneficiaryNo("XD201G0589941750");
        NextPlsDoBeneficiaryDelRequest beneficiaryDelRequest = 
                NextPlsDoBeneficiaryDelRequest.build(beneficiaryRequestDto);
        client.execute(beneficiaryDelRequest);
      
    }
}
```

### 5.3.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_BENEFICIARY_DEL | M
entity | | Object | Parameter list | M
| | clientBeneficiaryNo | String(20) | Unique code for partner beneficiary
| | beneficiaryNo | String(20) | Unique code for NextPls beneficiary

> Response Body:

```json
{
    "apiName": "DO_BENEFICIARY_DEL_R",
    "code": "200",
    "msg": "success",
    "entity": {}
}
```

### 5.3.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_BENEFICIARY_DEL_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list




## 5.4.GetBeneficiary
This method allows the partner to Get Registered Beneficiary Profile by client Beneficiary No or Beneficiary No
### 5.4.1.HTTP Request
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
    -H "Authorization: your authorization"
    -H "Signature: generated signature"
    -H "Content-Code: generated content-code"
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
        
        NextPlsClient client = 
            new DefaultNextPlsClient(
                "http://staging.nextpls.com/v1/remittance", 
                "test_client", "cek_tester_remit", "initial_tester01", 
                publicKey, secretKey);
        NextPlsBeneficiaryRequestDto beneficiaryRequestDto = new NextPlsBeneficiaryRequestDto();
        beneficiaryRequestDto.setClientBeneficiaryNo("TESTB002");
        NextPlsGetBeneficiaryRequest beneficiaryRequest = 
                NextPlsGetBeneficiaryRequest.build(beneficiaryRequestDto);
        client.execute(beneficiaryRequest);
      
    }
}
```

### 5.4.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_BENEFICIARY | M
entity | | Object | Parameter list | M
| | clientBeneficiaryNo | String(20) | Unique code for partner beneficiary | C
| | beneficiaryNo | String(20) | Unique code for NextPls beneficiary | C

> Response Body:

```json
{
    "apiName": "GET_BENEFICIARY_R",
    "code": "200",
    "msg": "success",
    "entity": {
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "XD201G0589941750",
        "firstName": "Beneficiary_First_Name",
        "middleName": "Beneficiary_Middle_Name",
        "lastName": "Beneficiary_Last_Name",
        "mobile": "12345678910",
        "gender": "M",
        "birthdate": "01/01/1994",
        "email": "nextPls@nextPls.com",
        "address1": "Philippines",
        "address2": "",
        "address3": "",
        "postalCode": "",
        "occupation": "",
        "idType": "PASSPORT",
        "idNumber": "PS256454165",
        "idDesc": "",
        "idIssueDate": "01/01/1994",
        "idExpDate": "01/01/1994",
        "nationality": "HKG",
        "relationship": "BROTHER",
        "accountNumber": "",
        "account": ""
    }
}
```

### 5.4.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_BENEFICIARY_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | clientBeneficiaryNo | String | Unique code for agent beneficiary
| | beneficiaryNo | String | Unique code for NextPls
| | firstName | String | Beneficiary First Name
| | middleName | String | Beneficiary Middle Name
| | lastName | String | Beneficiary Last Name
| | mobile | String | Mobile phone Number of Beneficiary
| | email | String | Email of Beneficiary
| | address1 | String | Beneficiary Address1
| | address2 | String | Beneficiary Address2
| | address3 | String | Beneficiary Address3
| | postalCode | String | Beneficiary Postal Code
| | occupation | String | Beneficiary Occupation
| | idType | String | Type of Beneficiary Id Proof
| | idNumber | String | Beneficiary ID Number
| | idDesc | String | Description
| | idIssueDate | String | Issue date(MM/DD/YYYY)
| | idExpDate | String | Expiry date(MM/DD/YYYY)
| | birthdate | String | Beneficiary BirthDate(MM/DD/YYYY)
| | gender | String | Gender
| | nationality | String | Nationality 
| | relationship | String | The relationship with the remitter 
| | bankCode | String | Bank code
| | accountNumber | String | Bank Account Number of Beneficiary
| | bankAccountName | String | Bank Account name of Beneficiary
| | account | String | Account of Beneficiary
| | bankAddress | String | Beneficiary Bank Address


# Transaction
## 6.1.GetBalance
This method allows the partner to Get the Balance by currency. 
### 6.1.1.HTTP Request
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
    "msg": "success",
    "entity": {
        "partnerCode": "test_client",
        "currency": "HKD",
        "balance": "1999.33"
    }
}
```

### 6.1.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_BALANCE_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | partnerCode | String | Partner Code
| | currency | String | Currency
| | balance | String | Balance

## 6.2.GetExRate
This method allows the partner to Get the last rate and lock one hour. 
### 6.2.1.HTTP Request
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
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_EX_RATE | M
entity | | Object | Parameter list | M
| | payInCountry | String(3) | Pay In Currency | M
| | payInCurrency | String(3) | Pay In Currency | M
| | payOutCountry | String(3) | Pay In Currency | M
| | payOutCurrency | String(3) | Pay Out Currency | M

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
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_EX_RATE_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | payInCountry | String | Pay In Currency
| | payInCurrency | String | Pay In Currency
| | payOutCountry | String | Pay In Currency
| | payOutCurrency | String | Pay Out Currency
| | exRate | String | The exchange rate


## 6.3.GetExRateLock
This method allows the partner to Get the last rate and locked
### 6.3.1.HTTP Request
<span class="http-method post">POST</span> `GET_EX_RATE_LOCK`

```json
{
    "apiName": "GET_EX_RATE_LOCK",
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
         "apiName": "GET_EX_RATE_LOCK",
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
        NextPlsGetExRateLockRequest exRateRequest = NextPlsGetExRateLockRequest.build(exRateDto);
        client.execute(exRateRequest);
      
    }
}
```

### 6.3.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_EX_RATE_LOCK | M
entity | | Object | Parameter list | M
| | payInCountry | String(3) | Pay In Currency | M
| | payInCurrency | String(3) | Pay In Currency | M
| | payOutCountry | String(3) | Pay In Currency | M
| | payOutCurrency | String(3) | Pay Out Currency | M

> Response Body:

```json
{
    "apiName": "GET_EX_RATE_LOCK_R",
    "code": "200",
    "msg": "success",
    "entity": {
        "payInCountry": "HKG",
        "payInCurrency": "HKD",
        "payOutCountry": "PHL",
        "payOutCurrency": "PHP",
        "exRate": "7.369781",
        "token": "dfb14532-ca6d-4f43-a5bc-045163b045ca"
    }
}
```

### 6.3.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_EX_RATE_LOCK_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | payInCountry | String | Pay In Currency
| | payInCurrency | String | Pay In Currency
| | payOutCountry | String | Pay In Currency
| | payOutCurrency | String | Pay Out Currency
| | exRate | String | The exchange rate
| | token | String | The token for locked



## 6.4.DoTransactionPre
This method allows the partner to preview the transfer details and keep the rate for some hours.
### 6.4.1.HTTP Request
<span class="http-method post">POST</span> `DO_TRANSACTION_PRE`

> Request Body:

```json
{
    "apiName": "DO_TRANSACTION_PRE",
    "entity": {
        "clientTxnNo": "1000",
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
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_PRE
entity | | Object | Parameter list
| | clientTxnNo | String(20) | Unique code for partner txn | M
| | payInCountry | String(3) | Pay In Country Code | M
| | payInCurrency | String(3) | Pay In Currency | M
| | payInAmount | String(18) | Pay In Amount | C
| | payOutCountry | String(3) | Pay Out Country Code | M
| | payOutCurrency | String(3) | Pay Out Currency | M
| | payOutAmount | String(18) | Pay Out Currency | C
| | transferCurrency | String(3) | Transfer Currency(The Fixed End, it must be payInCurrency or payOutCurrency) | M
| | paymentMode | String(20) | Payment Mode | M

> Response Body:

```json
{
    "apiName": "DO_TRANSACTION_PRE_R",
    "code": "200",
    "msg": "success",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000",
        "payInCurrency": "HKD",
        "payOutCurrency": "PHP",
        "payOutAmount": "100",
        "transferCurrency": "HKD",
        "paymentMode": "Bank",
        "exchangeRate": "7.369781",
        "commission": "0.1000",
        "totalAmount": "13.67"
    }
}
```

### 6.4.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_PRE_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | payInCurrency | String | Pay In Currency
| | payOutCurrency | String | Pay Out Currency
| | payOutAmount | String | Pay Out Amount
| | transferCurrency | String | Transfer Currency
| | paymentMode | String | Payment Mode
| | exchangeRate | String | Exchange Rate
| | commission | String | commission
| | totalAmount | String | Total Amount to pay

## 6.5.DoTransaction
This method allows the partner to initiate the transfer.
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
        "sourceIncome": "SALARY",
        
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
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_TRANSACTION | M
entity | | Object | Parameter list | M
| | txnNo | String(20) | Unique code for NextPls txn | M
| | clientTxnNo | String(20) | Unique code for partner txn | M
| | remitterNo | String(20) | Unique code for NextPls remitter | C
| | beneficiaryNo | String(20) | Unique code for NextPls beneficiary | C
| | purposeCode | String(16) | Purpose Code for txn | M
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
| | remitterPostalCode | String(16) | Remitter PostalCode | C
| | remitterOccupation | String(64) | Remitter Occupation | C
| | remitterIdType | String(16) | Remitter identity type | M
| | remitterIdNumber | String(20) | ID number | M
| | remitterIdDesc | String(30) | description for id | C
| | remitterIdIssueDate | String(10) | ID issue date (MM/DD/YYYY) | O
| | remitterIdExpDate | String(10) | ID expiry date (MM/DD/YYYY)  | O
| | remitterBirthdate | String(10) | Remitter date of birth (MM/DD/YYYY) | O
| | remitterGender | String(1) | Remitter gender. M=Male, F=Female | O
| | remitterNationality | String(3) | Remitter Nationality(3 characters Country ISO code) | M
| | remitterAccountNumber | String(30) | Remitter account number | O
| | sourceIncome | String(16) | Remitter source of income | M
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
| | beneficiaryPostalCode | String(16) | Beneficiary PostalCode | C
| | beneficiaryOccupation | String(64) | Beneficiary Occupation | C
| | beneficiaryIdType | String(16) | Type of Beneficiary Id Proof | O
| | beneficiaryIdNumber | String(20) | Beneficiary ID Number | O
| | beneficiaryIdDesc | String(20) | Description of Beneficiary ID | C
| | beneficiaryIdIssueDate | String(10) | Issue date(MM/DD/YYYY) | O
| | beneficiaryIdExpDate | String(10) | Expiry date(MM/DD/YYYY) | O
| | beneficiaryBirthdate | String(10) | Beneficiary BirthDate(MM/DD/YYYY) | O
| | beneficiaryGender | String(1) | Gender of Beneficiary | O
| | beneficiaryNationality | String(3) | Nationality of Beneficiary(3 Character Country ISO Code) | M
| | beneficiaryRelationship | String(16) | Relationship with the remitter | M
| | beneficiaryBankCode | String(20) | Bank code for Beneficiary | C
| | beneficiaryBankAccountNumber | String(30) | Bank Account Number of Beneficiary | C
| | beneficiaryBankAccountName | String(35) | Bank Account name of Beneficiary | C
| | beneficiaryAccount | String(35) | Account of Beneficiary | C
| | beneficiaryBankAddress | String(35) | Beneficiary Bank Address | O

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
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_TRANSACTION_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | status | String | The Transaction status


## 6.6.DoTokenTransaction
This method allows the partner to initiate the transfer by a rate token.
Before using this method, you must request the 'GetExRateLock'
### 6.6.1.HTTP Request
<span class="http-method post">POST</span> `DO_TOKEN_TRANSACTION`

> Request Body:

```json
{
    "apiName": "DO_TOKEN_TRANSACTION",
    "entity": {
       "token": "dfb14532-ca6d-4f43-a5bc-045163b045ca",
       "clientTxnNo": "1000",
       "purposeCode": "EDUCATION",
       "paymentMode": "Bank",
       "payInCountry": "HKG",
       "payInCurrency": "HKD",
       "payInAmount": "13.57",
       "payOutCountry": "PHL",
       "payOutCurrency": "PHP",
       "payOutAmount": "100",
       "transferCurrency": "PHP",
       
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
        "sourceIncome": "SALARY",
        
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
        "beneficiaryNationality": "PHL",
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
         "apiName": "DO_TOKEN_TRANSACTION",
         "entity": {
            "token": "dfb14532-ca6d-4f43-a5bc-045163b045ca",
            "clientTxnNo": "1000",
            "purposeCode": "EDUCATION",
            "paymentMode": "Bank",
            "payInCountry": "HKG",
            "payInCurrency": "HKD",
            "payInAmount": "13.57",
            "payOutCountry": "PHL",
            "payOutCurrency": "PHP",
            "payOutAmount": "100",
            "transferCurrency": "PHP",
            
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
             "sourceIncome": "SALARY",
             
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
             "beneficiaryNationality": "PHL",
             "beneficiaryRelationship": "BROTHER",
             "beneficiaryBankCode": "11003544",
             "beneficiaryBankAccountNumber": "4555556564564",
             "beneficiaryBankAccountName": "Benificiary_BankAccountName",
             "beneficiaryAccount": "",
             "beneficiaryBankAddress": ""
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
        txnRequestDto.setToken("dfb14532-ca6d-4f43-a5bc-045163b045ca");
        txnRequestDto.setClientTxnNo("1000");
        txnRequestDto.setPurposeCode("EDUCATION");
        txnRequestDto.setPaymentMode("Bank");
        txnRequestDto.setPayInCountry("HKG");
        txnRequestDto.setPayInCurrency("HKD");
        txnRequestDto.setPayInAmount("");
        txnRequestDto.setPayOutCountry("PHL");
        txnRequestDto.setPayOutCurrency("PHP");
        txnRequestDto.setPayOutAmount("1000");
        txnRequestDto.setTransferCurrency("PHP");
        
        txnRequestDto.setRemitterFirstName("ming");
        txnRequestDto.setRemitterMiddleName("");
        txnRequestDto.setRemitterLastName("xiao");
        txnRequestDto.setRemitterFirstLocalName("");
        txnRequestDto.setRemitterMiddleLocalName("");
        txnRequestDto.setRemitterLastLocalName("");
        txnRequestDto.setRemitterMobile("1234567890");
        txnRequestDto.setRemitterGender("F");
        txnRequestDto.setRemitterBirthdate("01/01/1994");
        txnRequestDto.setRemitterEmail("PandaRemit@gmail.com");
        txnRequestDto.setRemitterIdIssueDate("01/01/2020");
        txnRequestDto.setRemitterIdExpDate("10/10/2025");
        txnRequestDto.setRemitterNationality("HKG");
        txnRequestDto.setRemitterAddress1("HONG KONG");
        txnRequestDto.setRemitterAddress2("");
        txnRequestDto.setRemitterAddress3("");
        txnRequestDto.setRemitterPostalCode("");
        txnRequestDto.setRemitterOccupation("");
        txnRequestDto.setRemitterIdType("PASSPORT");
        txnRequestDto.setRemitterIdNumber("PS256454165");
        txnRequestDto.setRemitterIdDesc("");
        txnRequestDto.setRemitterAccountNumber("");
        txnRequestDto.setSourceOfIncome("SALARY");
        
        txnRequestDto.setBeneficiaryFirstName("hong");
        txnRequestDto.setBeneficiaryMiddleName("");
        txnRequestDto.setBeneficiaryLastName("xiao");
        txnRequestDto.setBeneficiaryFirstLocalName("");
        txnRequestDto.setBeneficiaryMiddleLocalName("");
        txnRequestDto.setBeneficiaryLastLocalName("");
        txnRequestDto.setBeneficiaryMobile("46176767667");
        txnRequestDto.setBeneficiaryEmail("");
        txnRequestDto.setBeneficiaryAddress1("Philippines");
        txnRequestDto.setBeneficiaryAddress2("");
        txnRequestDto.setBeneficiaryAddress3("");
        txnRequestDto.setBeneficiaryPostalCode("");
        txnRequestDto.setBeneficiaryOccupation("");
        txnRequestDto.setBeneficiaryIdType("PASSPORT");
        txnRequestDto.setBeneficiaryIdNumber("");
        txnRequestDto.setBeneficiaryIdDesc("");
        txnRequestDto.setBeneficiaryIdIssueDate("");
        txnRequestDto.setBeneficiaryIdExpDate("");
        txnRequestDto.setBeneficiaryBirthdate("");
        txnRequestDto.setBeneficiaryGender("");
        txnRequestDto.setBeneficiaryNationality("PHL");
        txnRequestDto.setBeneficiaryRelationship("BROTHER");
        txnRequestDto.setBeneficiaryBankCode("11003544");
        txnRequestDto.setBeneficiaryBankAccountNumber("4555556564564");
        txnRequestDto.setBeneficiaryBankAccountName("Benificiary_BankName");
        txnRequestDto.setBeneficiaryAccount("");
        txnRequestDto.setBeneficiaryBankAddress("");
        NextPlsDoSimpleTxnRequest simpleTxnRequest = NextPlsDoSimpleTxnRequest.build(txnRequestDto);
        client.execute(simpleTxnRequest);
      
    }
}
```

### 6.6.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_TOKEN_TRANSACTION | M
entity | | Object | Parameter list | M
| | token | String(36) | Unique token for locked rate | M
| | clientTxnNo | String(20) | Unique code for partner txn | M
| | purposeCode | String(16) | Purpose Code for txn | M
| | paymentMode | String(20) | Payment Mode | M
| | payInCountry | String(3) | Pay In Country | M
| | payInCurrency | String(3) | Pay In Currency | M
| | payInAmount | String(18) | Pay In Amount | C
| | payOutCountry | String(3) | Pay Out Country | M
| | payOutCurrency | String(3) | Pay Out Currency | M
| | payOutAmount | String(18) | Pay Out Amount | C
| | transferCurrency | String(3) | Transfer Currency | M
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
| | remitterPostalCode | String(16) | Remitter PostalCode | C
| | remitterOccupation | String(64) | Remitter Occupation | C
| | remitterIdType | String(16) | Remitter identity type | M
| | remitterIdNumber | String(20) | ID number | M
| | remitterIdDesc | String(30) | description for id | C
| | remitterIdIssueDate | String(10) | ID issue date (MM/DD/YYYY) | O
| | remitterIdExpDate | String(10) | ID expiry date (MM/DD/YYYY)  | O
| | remitterBirthdate | String(10) | Remitter date of birth (MM/DD/YYYY) | O
| | remitterGender | String(1) | Remitter gender. M=Male, F=Female | O
| | remitterNationality | String(3) | Remitter Nationality(3 characters Country ISO code) | M
| | remitterAccountNumber | String(30) | Remitter account number | O
| | sourceIncome | String(16) | Remitter source of income | M
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
| | beneficiaryPostalCode | String(16) | Beneficiary PostalCode | C
| | beneficiaryOccupation | String(64) | Beneficiary Occupation | C
| | beneficiaryIdType | String(16) | Type of Beneficiary Id Proof | O
| | beneficiaryIdNumber | String(20) | Beneficiary ID Number | O
| | beneficiaryIdDesc | String(20) | Description of Beneficiary ID | C
| | beneficiaryIdIssueDate | String(10) | Issue date(MM/DD/YYYY) | O
| | beneficiaryIdExpDate | String(10) | Expiry date(MM/DD/YYYY) | O
| | beneficiaryBirthdate | String(10) | Beneficiary BirthDate(MM/DD/YYYY) | O
| | beneficiaryGender | String(1) | Gender of Beneficiary | O
| | beneficiaryNationality | String(3) | Nationality of Beneficiary(3 Character Country ISO Code) | M
| | beneficiaryRelationship | String(16) | Relationship with the remitter | M
| | beneficiaryBankCode | String(20) | Bank code for Beneficiary | C
| | beneficiaryBankAccountNumber | String(30) | Bank Account Number of Beneficiary | C
| | beneficiaryBankAccountName | String(35) | Bank Account name of Beneficiary | C
| | beneficiaryAccount | String(35) | Account of Beneficiary | C
| | beneficiaryBankAddress | String(35) | Beneficiary Bank Address | O

> Response Body:

```json
{
    "apiName": "DO_TOKEN_TRANSACTION_R",
    "code": "200",
    "msg": "success",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000",
        "status": "TRANSACTION_ING"
    }
}
```

### 6.6.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_TOKEN_TRANSACTION_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | status | String | The Transaction status


## 6.7.GetTransactionStatus
This method allows the partner to check the Transaction status. 
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
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | GET_TRANSACTION_STATUS | M
entity | | Object | Parameter list | M
| | txnNo | String(20) | Unique code for NextPls txn | M
| | clientTxnNo | String(20) | Unique code for partner txn | M

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

### 6.7.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_TRANSACTION_STATUS_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | reference | String | Unique code for cash withdrawal
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | status | String | The Transaction status



## 6.8.DoTransactionCancel
This method allows the partner to chancel the Transaction, but just support the cash transaction. 
### 6.8.1.HTTP Request
<span class="http-method post">POST</span> `DO_TRANSACTION_CANCEL`

> Request Body:

```json
{
    "apiName": "DO_TRANSACTION_CANCEL",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000",
        "reason": "I don't want to remit anymore."
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
         "apiName": "DO_TRANSACTION_CANCEL",
         "entity": {
             "txnNo": "IU201G0279816077",
             "clientTxnNo": "1000",
             "reason": "I don't want to remit anymore."
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
        NextPlsTransactionCancelRequestDto cancelRequestDto = new NextPlsTransactionCancelRequestDto();
        cancelRequestDto.setTxnNo("IU201G0279816077");
        cancelRequestDto.setClientTxnNo("1000");
        cancelRequestDto.setReason("I don't want to remit anymore.");
        NextPlsDoTransactionCancelRequest txnCancelRequest = NextPlsDoTransactionCancelRequest.build(cancelRequestDto);
        client.execute(txnCancelRequest);
      
    }
}
```

### 6.8.2.Request Body
Field |  | Type | Describe | O/M
--------- | ------- | ------- | ---------- | -------
apiName | | String | DO_TRANSACTION_CANCEL | M
entity | | Object | Parameter list | M
| | txnNo | String(20) | Unique code for NextPls txn | M
| | clientTxnNo | String(20) | Unique code for partner txn | M
| | reason | String(64) | the reason for refund | M

> Response Body:

```json
{
    "apiName": "DO_TRANSACTION_CANCEL_R",
    "code": "200",
    "msg": "success",
    "entity": {
        "txnNo": "IU201G0279816077",
        "clientTxnNo": "1000",
        "status": "PROCESSING_CANCELING"
    }
}
```

### 6.8.3.Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_TRANSACTION_STATUS_R
code | | String | Result Code
msg | | String | Result message
entity | | Object | Parameter list
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | status | String | The Transaction status



# Errors
## 7.1.HTTP Error Codes
### These are error codes that will be returned in the body of API responses

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

