---
title: API PandaRemit

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

# Version 1.0.0
[简体中文版](../)
## Introduction

Welcome the API document for NextPls!

# Getting Started
## Cryptography in NextPls API 
### Request

### 1.Generating a CEK

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
  
### 2.Encrypting body part with the CEK

>An example of encrypting the body:

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

### 3.Encrypting CEK with NextPls public key

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
  
### 4.Generating signature

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
  
### 5.Generating request header and body

With encrypted CEK(step.3), encrypted body(step.2) and HMAC(step.4) values, Agent can generate a http header like followings;  
  
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

### Response

### 1.Verifying signature

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

### 2.Deriving CEK

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

### 3.Decrypting body part with CEK

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

## Keys

Definition of abbreviated values found in the documentation.

Parameter | Type | Description
--------- | ------- | -----------
M | string | value for this field is required
O | string | value for this field is optional
C | string | The requirement for this field is conditional based on other field values

# Basic Information
## GetCountryPayMode
This method allows the partner to Get Country based Payment Modes.
### HTTP Request
<span class="http-method post">POST</span> `GET_COUNTRY_PAY_MODE`

The function is not open yet, Coming soon!

# Remitter

## DoRemitterAdd
This method allows the partner to Registered Remitter Profile. 
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
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

### Request Body
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
| | idType | int(3) | Remitter identity type | M
| | idNumber | String(20) | ID number | M
| | idDesc | String(30) | description when ID Type=6 | C
| | idIssueDate | String(10) | ID issue date (MM/DD/YYYY) | O
| | idExpDate | String(10) | ID expiry date (MM/DD/YYYY)  | O
| | birthdate | String(10) | Remitter date of birth (MM/DD/YYYY) | O
| | sex | String(1) | Remitter gender. M=Male, F=Female | O
| | nationality | String(3) | Remitter Nationality(3 characters Country ISO code) | M
| | accountNumber | String(30) | Remitter account number | O
| | sourceIncome | String(2) | Remitter source of income | M

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

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_REMITTER_ADD_R
code | | String | Result Code
entity | | Object | Parameter list
| | clientRemitterNo | String | Unique code for partner remitter
| | remitterNo | String | Unique code for NextPls remitter
msg | | String | Result message

## DoRemitterEdit
This method allows the partner to Edit Registered Remitter Profile.
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
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

### Request Body
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
| | idType | int(3) | Remitter identity type | O
| | idNumber | String(20) | ID number | O
| | idDesc | String(30) | description when ID Type=6 | O
| | idIssueDate | String(10) | ID issue date (MM/DD/YYYY) | O
| | idExpDate | String(10) | ID expiry date (MM/DD/YYYY)  | O
| | birthdate | String(10) | Remitter date of birth (MM/DD/YYYY) | O
| | sex | String(1) | Remitter gender. M=Male, F=Female | O
| | nationality | String(3) | Remitter Nationality(3 characters Country ISO code) | O
| | accountNumber | String(30) | Remitter account number | O
| | sourceIncome | String(2) | Remitter source of income | O

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

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_REMITTER_EDIT_R
code | | String | Result Code
entity | | Object | Parameter list
| | clientRemitterNo | String | Unique code for partner remitter
| | remitterNo | String | Unique code for NextPls remitter
msg | | String | Result message


## GetRemitter
This method allows the partner to Get Registered Remitter Profile.  
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
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

### Request Body
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
    "entity": {
        "clientRemitterNo": "TEST_B001",
        "remitterNo": "JJ201A1131599873",
        "firstName": "REMITTER_First_Name",
        "middleName": "REMITTER_Middle_Name",
        "lastName": "REMITTER_Last_Name",
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
| | idType | int | Remitter identity type
| | idNumber | String | ID number
| | idDesc | String | description
| | idIssueDate | String | ID issue date (MM/DD/YYYY)
| | idExpDate | String | ID expiry date (MM/DD/YYYY)
| | birthdate | String | Remitter date of birth (MM/DD/YYYY)
| | sex | String | Remitter gender
| | nationality | String | Remitter Nationality
| | accountNumber | String | Remitter account number
| | sourceIncome | String | Remitter source of income
msg | | String | Result message






# Beneficiary

## DoBeneficiaryAdd
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


## DoBeneficiaryEdit
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


## DoBeneficiaryDel
This method allows the partner to Delete Registered beneficiary.
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
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

### Request Body
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
    "entity": {},
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | DO_BENEFICIARY_DEL_R
code | | String | Result Code
entity | | Object | Parameter list
msg | | String | Result message






## GetBeneficiary
This method allows the partner to Get Registered Beneficiary Profile by client Beneficiary No or Beneficiary No
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
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
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

### Request Body
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
    "entity": {
        "clientBeneficiaryNo": "BE1233112",
        "beneficiaryNo": "XD201G0589941750",
        "firstName": "Beneficiary_First_Name",
        "middleName": "Beneficiary_Middle_Name",
        "lastName": "Beneficiary_Last_Name",
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
        "relationship": "3",
        "accountNumber": ""
    },
    "msg": "success"
}
```

### Response Body
Field |   | Type | Describe
--------- | ------- | ------- |-----------
apiName | | String | GET_BENEFICIARY_R
code | | String | Result Code
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
| | idType | int | Type of Beneficiary Id Proof
| | idNumber | String | Beneficiary ID Number
| | idDesc | String | Description
| | idIssueDate | String | Issue date(MM/DD/YYYY)
| | idExpDate | String | Expiry date(MM/DD/YYYY)
| | birthdate | String | Beneficiary BirthDate(MM/DD/YYYY)
| | sex | String | Gender
| | nationality | String | Nationality 
| | relationship | String | The relationship with the remitter 
| | bankCode | String | Bank code
| | accountNumber | String | Bank Account Number of Beneficiary
| | bankAccountName | String | Bank Account name of Beneficiary
| | bankAddress | String | Beneficiary Bank Address
msg | | String | Result message


# Transaction
## GetBalance
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

## GetExRate
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

## DoTransactionPre
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

## DoTransaction
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
       "remitterNo": "JJ201A1131599873",
       "beneficiaryNo": "RP122141",
       "purposeCode": "3"
    }
}
```
```shell
curl -X POST http://staging.nextpls.com/v1/remittance
    -H "Content-Type: application/base64"
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
| | remitterNo | String(20) | Unique code for NextPls remitter | M
| | beneficiaryNo | String(20) | Unique code for NextPls beneficiary | M
| | purposeCode | String(2) | Purpose Code for txn | M

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

## GetTransactionStatus
This method allows the partner to check the Transaction status. 
### HTTP Request
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
    -H ”Authorization:"your authorization"
    -H "Signature:"generated signature"
    -H "Content-Code:"generated content-code"
    -d
    '{
         "apiName": "GET_TRANSACTION_STATUS",
         "entity": {
             "txnNo": "IU201G0279816077",
             "clientTxnNo": "1000"
         }
     }'
```

### Request Body
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
apiName | | String | GET_TRANSACTION_STATUS_R
code | | String | Result Code
entity | | Object | Parameter list
| | txnNo | String | Unique code for NextPls txn
| | clientTxnNo | String | Unique code for partner txn
| | status | String | The Transaction status
msg | | String | Result message


# Errors
## HTTP Error Codes
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
