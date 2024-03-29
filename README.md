##StageGig API

## <a name='TOC'>Table of Contents</a>

1. [General Informations & Assumptions](#general)
2. [Vendor Login & Token Retriaval](#login)
3. [Registration Steps](#registration)

## <a name="general"> General Informations & Assumptions </a>

* All request headers should contain;
  * Content-Type: application/json
  * Authentication: "Bearer TokenValue"  (except for login)
   
* All routes have **/api/v1/** default prefix.

* Optional parameters end with **(?)**
 
* If error occurs every endpoints returns ```error``` object, in any case ***you must check*** top-level error object;

```js
	
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
```

* Each endpoints, takes ***info*** object, 

```js
	{
		"info": {
			"language": "TR", // Two Character Universal Country Code, REST şu an sadece TR destekliyor.
			"device": "WEB" // WEB|IOS|ANDROID (All uppercased)
		},
		// Rest of json
		
	}
```

* If token expires or not valid, rest server replies with **error.code** is **8002** , and you must retrieve another token from rest.


**Error Codes:** 

8000-8999 -> App related errors, user does not meant to be see that.

8001 -> Not a valid json structure

8101 -> Token Expiration
8102 -> Invalid Token

8201 -> Database Error
8202 -> Mailer Error ( Invite & Verify mails )

8301 -> Internal Server Error

8401 -> Not authorized to call this api

9000-9999 -> User related errors, user **meant to see** that
9001 -> User input errors ( ex: not valid email ) ( according to language rest return pretty message )

```js
	// If error code starts with "8" do not show related msg to user,
	// If starts with "8" print related msg.
	{
		"error": {
			"code": 8002
			"msg": "Expired token."
		}
	}
```


## <a name="login"> Login/Token Retriaval </a>

> Dont forget to send ***info*** object even if token empty now!

> Web App do not need this section for now, 
> Token, InviteCode, Mail, Name(?), Surname(?) automatically generated



* **/vendor/login**
    * Payload:
	
	```js 
	{
        "info": {},
		"mail": "vendor@mail.com",
		"password": "232421"
	}
	```
    * Response:
	  
	```js 
	{
		"token": "3198391dklandjka837341"
	}
	```


* **/vendor/invite/login**

> Dont forget to send ***info*** object!

* Payload:
	
	```js 
	{
	    "info": {},
		"mail": "vendor@mail.com",
		"invite_code": 91356 // 5 digit integer
	}
	```
* Response:
	  
	```js 
	{
		"token": "3198391dklandjka837341",
		"step": 1|2|3,
		"vendor": {
			"vendor_mail":"vendor@mail.com",
			"vendor_name":"name", // optional
			"vendor_surname":"surname", // optional
		} 
	}
	```
* Example:

```js
{
  "mail": "darkalive@gmail.com",
  "invite_code":"18667",
  "info": {
    "language": "EN",
    "device": "WEB"
  }
}

{		     "token":"ODkwY2FlYjUtOTVhNS00NzRmLWFhNjAtYTNlOTE3ODg1NDAwOkZGMTFDRENDLUQzQTctNEIyMi1CNUM4LTVBNjM1Njk3NzI0MA=="
} 
```
## <a name="registration"> Registration Steps </a>


* **/vendor/register/step1**

> Automatically pre-fill Vendor Mail and name,surname if exist.
> 
> Vendor can change pre-defined values including mail.
> 
> If vendor changes its mail, rest replies accordingly


  * Payload:
	
	```js 
	{
		"info": {},
		"step1": {
			"vendor_mail":"mail", // must be valid mail
			"vendor_password":"password", // must be at least 6 character
			"vendor_name":"name", // must be at least 2 character
			"vendor_surname":"surname", // must be at least 2 character
			"vendor_invite_code":"invite_code" // must be equal 5 character
			"vendor_subscribes": true|false // optional defaults to false
		}
	}
	```
  * Response:
	    
	```js 
	// If mail adress changed, already mail sent to user.
	{
		"action": {
			"open": "mail",
			"new_mail": "new@mail.com",
			"old_mail": "old@mail.com"
		}
	}
	```

	```js 
	// If mail adress does not changed
	{
		"action": {
			"open": "2"
		}
	}
	```

  * **/vendor/register/verify-mail**

>User can also click directly from the mail (mobile app_redirect ?)

  * Payload:
	
	```js 
	{
		"info": {},
		"mail_verification": {
			"verification_code": "57126"
		}
	}
	```
  * Response:
	    
	```js 
	{
		"action": {
			"open": "2"
		}
	}
	// OR - If error not occured, it means succesfully verified
	```js 
	{
        "error": {
            "code": 9001
            "msg": "Invalid code, please try again."
        }
    }
	```
  * **/vendor/register/send-verify-mail**

  * Payload:
	
	```js 
	{
		"info": {} // Info object enough, I know who the vendor is from token.
	}
	```
  * Response:
	    
	```js 
	{
		"action": {
            "msg": "Onay mailiniz tekrar gönderilmiştir"
		}
	}
	```
		// OR - If error not occured, it means mail have sent succesfully.
	```js 
	{
        "error": {
            "code": 9001
            "msg": "Please, wait 57 seconds to send again."
        }
    }
	```
	
  * **/vendor/register/step2**


  * Payload:

	```js
	{
		"info": {},
		"step2: {
            "vendor_id_number": "tcklimlikno",
            "vendor_gender":"male|female", // only "male" and "female" allowed
            "vendor_birthday": "06/07/1987", // only DD/MM/YYYY allowed
            "vendor_birthplace":"Bandırma",
            "vendor_phone_number":"+90 (541) 625 89 25", // only +90 (XXX) XXX XX XX allowed
        }
	}
	```
  * Response:
	    
	```js
	{
		"action": {
			"open": "sms",
            "vendor_phone_number": "+90 (541) 625 89 25"
		}
	}
	```



  * **/vendor/register/sms-verification**


  * Payload:

	```js
	{
		"info": {},
		"sms_verification: {
            "verification_code": "57126"
        }
	}
	```
  * Response:
	    
	```js
	{
		"action": {
			"open": "3"
		}
	}
	```
		// OR - If error not occured, it means succesfully verified
	```js 
	{
        "error": {
            "code": 9001
            "msg": "Invalid code, please try again."
        }
    }
	```
	
	
  * **/vendor/register/send-sms-verification**


  * Payload:

	```js
	{
		"info": {} // info yeterli
	}
	```
  * Response:
	    
	```js
	{
		"action": {
            "msg": "Onay kodunuzu içeren SMS telefonunuza tekrar gönderilmiştir"
		}
	}
	```
	    // OR - If error not occured, it means mail have sent succesfully.
	```js 
	{
        "error": {
            "code": 9001
            "msg": "Please, wait 57 seconds to send again."
        }
    }
	```


	
