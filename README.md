##StageGig API

## <a name='TOC'>Table of Contents</a>

1. [General Informations & Assumptions](#general)
2. [Vendor Login & Token Retriaval](#login)
3. [Registration Steps](#registration)

## <a name="general"> General Informations & Assumptions </a>

* All routes have **/api/v1/** default prefix.

* Optional parameters end with **(?)**
 
* If any error occurs in all endpoints returns ```error``` object, in any case ***you must check*** top-level error object;

````js
	
	{
		"error": {
			"code": Integer(5 digits)
			"msg": "Geçersiz mail adresi ve/veya davetiye kodu girdiniz, lütfen tekrar kontrol ediniz."
		}
	}
	
	---
	
	if(reponseJson.error != null) {
		// Do sth with error
	}
````

* Each endpoints except for login and token retrievals, takes ***info*** object, 

````js
	{
		"info": {
			"credentials": {
				"mail": "vendor@mail.com",
				"token": "generatedToken"
			},
			"language": "TR", // Two Character Universal Country Code, REST şu an sadece TR destekliyor.
			"device": "WEB" // WEB|IOS|ANDROID (All uppercased)
		},
		// Rest of json
		
	}
````

* If token expires or not valid, rest server replies with **error.code** is ***9001*** , and you must retrieve another token from rest.

````js
	// Except for token expiration, (after you check "error.code" != 9001) you can skip 
	// and print/show "error.msg" to user.
	{
		"error": {
			"code": 9001
			"msg": "Expired token."
		}
	}
````


## <a name="login"> Login/Token Retriaval </a>

> Web App do not need this section for now, 
> Token, InviteCode, Mail, Name(?), Surname(?) automatically generated


  * ###**/vendor/login**
    * Payload:
	
	````js 
	{
		"mail": "vendor@mail.com",
		"password": "232421"
	}
	````
    * Response:
	  
	````js 
	{
		"token": "3198391dklandjka837341"
	}
	````


* ###**/vendor/invite/login**
    * Payload:
	
	````js 
	{
		"mail": "vendor@mail.com",
		"invite_code": 91356 // 5 digit integer
	}
	````
    * Response:
	  
	````js 
	{
		"token": "3198391dklandjka837341",
		"step": 1|2|3|4,
		"vendor": {
			"vendor_mail":"vendor@mail.com",
			"vendor_name":"name", // optional
			"vendor_surname":"surname", // optional
		} 
	}
	````

## <a name="registration"> Registration Steps </a>

* ###**/vendor/register/step1**

> Automatically pre-fill Vendor Mail and name,surname if exist.
> 
> Vendor can change pre-defined values including mail.
> 
> If vendor changes its mail, rest replies accordingly


  * Payload:
	
	````js 
	{
		"info": {},
		"step1": {
			"vendor_mail":"mail", 
			"vendor_password":"password",
			"vendor_name":"name",
			"vendor_surname":"surname",
		}
	}
	````
  * Response:
	    
	````js 
	// If mail adress changed
	{
		"action": {
			"open": "mail",
			"new_mail": "new@mail.com",
			"old_mail": "old@mail.com"
		}
	}
	````

	````js 
	// If mail adress does not changed
	{
		"action": {
			"open": "2"
		}
	}
	````


* ###**/vendor/register/mail_verification**

>User can also click directly from the mail (mobile app_redirect ?)

  * Payload:
	
	````js 
	{
		"info": {},
		"mail_verification": {
			"verification_code": "57126"
		}
	}
	````
  * Response:
	    
	````js 
	{
		"action": {
			"open": "2"
		}
	}
	````

* ###**/vendor/register/step2**


  * Payload:

	````js
	{
		"info": {},
		"step2: {
            "vendor_id_number": "tcklimlikno",
            "vendor_gender":"male|female", // only "male" and "female" allowed
            "vendor_birthday": "06/07/1987", // only DD/MM/YYYY allowed
            "vendor_birthplace":"Bandırma",
            "vendor_phone_number":"+90 (541) 625 89 25", // only +90 (XXX) XXX XX XX allowed
            "":"",
            "":"",
        }
	}
	````
  * Response:
	    
	````js
	{
		"action": {
			"open": "sms",
            "vendor_phone_number": "+90 (541) 625 89 25"
		}
	}
	````



* ###**/vendor/register/sms-verification**


  * Payload:

	````js
	{
		"info": {},
		"sms_verification: {
            "verification_code": "57126"
        }
	}
	````
  * Response:
	    
	````js
	{
		"action": {
			"open": "3"
		}
	}
	````
	
* ###**/vendor/register/send-sms-verification**


  * Payload:

	````js
	{
		"info": {} // info yeterli
	}
	````
  * Response:
	    
	````js
	{
		"action": {
			"msg": "Yeni sms gönderimi yapabilmek için 60 saniye daha beklemelisiniz",
            ||
            "msg": "Onay kodunuzu içeren SMS telefonunuza tekrar gönderilmiştir"
		}
	}
	````


	
