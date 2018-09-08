# Android SDK 0.7

## Getting started

### Account Setup

To ensure the authenticity of the interactions between your app and Truecaller, you need to generate a partner key from the truecaller developer portal ( https://developer.truecaller.com/auth/login ) by providing us with your package name and SHA-1 signing-fingerprint.

You package name corresponds to the applicationId in your app level build.gradle file. Use the following command to get the fingerprint:

```java
keytool -list -v -keystore mystore.keystore
```

Once we have received the package name and the SHA-1 signing-fingerprint, we will provide you with a unique "PartnerKey" which you need to include in your project to authorize all verification requests.

### Understanding how the user verification flow works

#### A. Truecaller app present on device

 - User starts by clicking on your Login / Signup CTA and would be shown the standard Truecaller consent popup asking for the user's consent. The user authorizes by clicking continue and their Truecaller profile would be shared with you

#### B. Truecaller app not present on device

 - User verifying on an app using Truecaller SDK for the first time using his mobile number on a device :
![Diagram](https://github.com/singhalyogesh/sdk-doc/blob/master/1.png)
 - User verifying on an app using Truecaller SDK when he has already been verified earlier using his mobile number on that device in the same app or any other app present on the device :
![Diagram](https://github.com/singhalyogesh/sdk-doc/blob/master/2.png)
 - User verifying on an app using Truecaller SDK using either a different phone number on the same device or using the same number on a different device :
![Diagram](https://github.com/singhalyogesh/sdk-doc/blob/master/3.png)


### Using the SDK with your Android Studio Project

1. Ensure that your Minimum SDK version is atleast API level 16 or above ( Android 4.1 ). In case your android project compiles for API level below 16, you can include the following line in your AndroidManifest.xml file to avoid any compilation issues :
```java
<uses-sdk tools:overrideLibrary="com.truecaller.android.sdk"/>
```
Using this would ensure that the sdk works normally for API level 16 & above, and would be disabled for API level < 16

2. Add the provided truesdk-0.7-releasePartner.aar file into your libs folder. Example path: /app/libs/ 
3. Open the build.gradle of your application module and ensure that your lib folder can be used as a repository :

    ```java
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
    ```
    
    Secondly add the compile dependency with the latest version of the TrueSDK aar :

    ```java
    dependencies {
        implementation(name: "truesdk-0.7-releasePartner", ext: "aar")
    }
    ```
    Add the following dependencies within your gradle file :
    ```java
    implementation 'com.squareup.retrofit2:retrofit:2.3.0'
    implementation 'com.squareup.okhttp3:okhttp:3.7.0'
    ```

4. Open your strings.xml file. Example path: /app/src/main/res/values/strings.xml and add a new string with the name "partnerKey" and value as your "PartnerKey"

5. Open your AndroidManifest.xml and add a meta-data element to the application element:
 
    ```java
    <application android:label="@string/app_name" ...>
    ...
    <meta-data android:name="com.truecaller.android.sdk.PartnerKey" android:value="@string/partnerKey"/>
    ...
    </application>
    ```

6. Add the TrueButton view in the selected layout. You can have only one TrueButton per Activity

    ```java
    
    <com.truecaller.android.sdk.TrueButton
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    truesdk:truebutton_text="cont"/>
    ```

7. Create a TrueSdkScope object by using the appropriate configurational settings and use it to initialize the TrueSDK in your android activity's onCreate method:
    
     ```java
     TrueSdkScope trueScope = new TrueSdkScope.Builder(this, sdkCallback)
		.sdkOptions( TrueSdkScope.SDK_OPTION_WITH_OTP )
		.consentMode(TrueSdkScope.CONSENT_MODE_FULLSCREEN )
		.consentTitleOption( TrueSdkScope.SDK_CONSENT_TITLE_VERIFY )
		.footerType( TrueSdkScope.FOOTER_TYPE_SKIP )
		.build();
                
     TrueSDK.init(trueScope);	
     ```
    
    TrueSDK v0.7 provides you with capabilities to configure the following settings -

    ##### - SDK Options
	To use the SDK for verification of only Truecaller users or to use it to verify all your users ( including non truecaller users as well )
	
	Possible values for SDK Options :
	
	```java      
	// SDK with verification capability for only Truecaller users who have the truecaller app present on the device
	TrueSdkScope.SDK_OPTION_WITH_OTP         
	
	// SDK with OTP verification functionality for all users, including non truecaller users ( via OTP verification )
	TrueSdkScope.SDK_OPTION_WITHOUT_OTP
	```

    ##### - Consent Mode 
	To switch between a full screen view or an overlay view of the profile consent
	
	Possible values for Consent Mode :
	
	```java
	// To display the user's Truecaller profile in a popup view
	TrueSdkScope.CONSENT_MODE_POPUP
	
	// To display the user's Truecaller profile in a full screen view
	TrueSdkScope.CONSENT_MODE_FULLSCREEN
	```
	
    ##### - Footer Type
	To configure the CTA present at the bottom
	
	Possible values for Footer Type :
	
	```java
	// To use "USE DIFFERENT NUMBER" CTA at the bottom of the user profile view
	TrueSdkScope.FOOTER_TYPE_CONTINUE
	
	// To use "SKIP" CTA at the bottom of the user profile view
	TrueSdkScope.FOOTER_TYPE_SKIP
	```
	
    ##### - Consent Title Options
	To provide appropriate context of verification to the truecaller user 
	
	Possible values for Consent Title option :
	
	```java
	// To use "Login" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_LOG_IN
	
	// To use "Signup" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_SIGN_UP
	
	// To use "Sign in" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_SIGN_IN
	
	// To use "Verify" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_VERIFY
	
	// To use "Register" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_REGISTER
	
	// To use "Get Started" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_GET_STARTED
	```

 8. (Optional) 
    You can set a unique requestID for every profile request with
    `TrueSDK.getInstance().setRequestNonce(customHash);`
    
    Note : The customHash must be a base64 URL safe string with a minimum character length of 8 and maximum of 64 characters

 9. Initialise the TrueButton in the onCreate method:

      ```java
      TrueButton trueButton = findViewById(R.id.com_truecaller_android_sdk_truebutton);
      ```
    
 10. Add the following condition in the onActivityResult method:

      ```java
      TrueSDK.getInstance().onActivityResultObtained( this,resultCode, data);
      ```
      
 11. In your selected Activity

   - Either make the Activity implement ITrueCallback or create an instance. 
	This interface has 3 methods: onSuccesProfileShared(TrueProfile), onFailureProfileShared(TrueError) and onOtpRequired()
   
   ```java
       private final ITrueCallback sdkCallback = new ITrueCallback() {
       
        @Override
        public void onSuccessProfileShared(@NonNull final TrueProfile trueProfile) {
	
		// This method is invoked when either the truecaller app is installed on the device and the user gives his
		// consent to share his truecaller profile OR when the user has already been verified before on the same
		// device using the same number and hence does not need OTP to verify himself again. 
	
		Log.d( TAG, "Verified without OTP! (Truecaller User): " + trueProfile.firstName );
        }

        @Override
        public void onFailureProfileShared(@NonNull final TrueError trueError) {
		// This method is invoked when some error occurs or if an invalid request for verification is made 
	
		Log.d( TAG, "onFailureProfileShared: " + trueError.getErrorType() );
        }

        @Override
        public void onOtpRequired() {
	
		// This method is invoked when truecaller app is not present on the device or if the user wants to
		// continue with a different number and hence, OTP verification is required to complete the flow
		// You can initiate the OTP verification flow from within this callback method by using :
	   
		TrueSDK.getInstance().requestVerification("IN", PHONE_NUMBER_STRING, apiCallback);
	    
		//  Here, the first parameter is the country code of the mobile number on which the OTP needs to be
		// triggered and PHONE_NUMBER_STRING should be the 10-digit mobile number of the user
	
        }
    };
    
   ```
    
   Write all the relevant logic in onSuccesProfileShared(TrueProfile) for displaying the information you have just received and onFailureProfileShared(TrueError) for handling the error and notify the user.
   
   Similarly, make your Activity implement OtpCallback or create an instance ( Once the OTP verification is triggered using the 'requestVerification' method, the control would then be passed to OtpCallback ) .
	This interface has 2 methods: onOtpSuccess(int, Bundle) and onOtpFailure(int, TrueException)
   
   ```java
       static final OtpCallback apiCallback = new OtpCallback() {

        @Override
        public void onOtpSuccess(int requestCode, @Nullable String s) {
		if (requestCode == OtpCallback.MODE_OTP_SENT) {
	    
	    		// This method is invoked when the OTP has been sent to the input mobile number.
			// You can now ask the user to input the 6-digit OTP code sent to him via SMS and ask for his
			// first name and last name
	    
			Log.d( TAG, "OTP Sent" );
		
		} else if ( requestCode == OtpCallback.MODE_VERIFIED ) {
	    
			// This method is invoked when the user has successfully input the correct OTP code along with his
			// first name and last name and is verified successfully by the SDK
	    
			Log.d( TAG, "Verified with OTP" );
		}
        }

        @Override
        public void onOtpFailure(final int requestCode, @NonNull final TrueException e) {
	    
		// Invoked when some error has occured while verifying the provided mobile number via OTP
		Log.d( TAG, "OnFailureApiCallback: " + e.getExceptionMessage() );
        }
    };
    
   ```
         
   The SMS sent by Truecaller would be in a below mentioned standard format containing a 6-digit numeric code along with your app name ( as configured by you while creating the app key using your Truecaller developer portal account ). You can use this format to auto-read and fill the OTP code in your app activity
   ```
   <<123456>> is your verification code for <<AppName>> (Verification Powered by Truecaller)
   ```
   
   
   To complete the verification once the OTP has been sent to the provided mobile number, complete the verification process by calling the following method from within your activity :
   
   ```
   TrueProfile profile = new TrueProfile.Builder(firstName, lastName).build();
   TrueSDK.getInstance().verifyOtp(profile, otp, apiCallback);
   
   // 'verifyOtp' method returns the control to the OtpCallback as defined in the section above
   ```

  (Optional)  
  In order to use a custom button instead of the default TrueButton call trueButton.onClick(trueButton) in its onClick listner. Make sure your button follow our visual guidelines.

### Advanced and Optional

#### A. Server side Truecaller Profile authenticity check [ for users who verified via Truecaller app consent flow ]

Inside TrueProfile class there are 2 important fields, payload and signature. Payload is a Base64 encoding of the json object containing all profile info of the user. Signature contains the payload's signature. You can forward these fields back to your backend and verify the authenticity of the information.


For details on the verification flow and sample code snippets, please refer the following link :
https://github.com/singhalyogesh/truesdk-backend-validation

IMPORTANT: Truecaller SDK already verifies the authenticity of the response before forwarding it to the your app.

#### B. Server side Truecaller Profile authenticity check [ for users who verified via Truecaller OTP flow ]

In OnSuccess method of OtpCallback, in case of MODE_VERIFIED, you can fetch the access token string. You can forward this field back to your backend and verify the authenticity of the information by using the following endpoint:

**Endpoint:**  
https://api4.truecaller.com/v1/otp/installation/validate/{accessToken}

**Method:**  
POST

**Header parameters:**

| **Parameter [Type]** | **Required** | **Description**         | **Example**                            |
| -------------------  | ------------ | ----------------------- | -------------------------------------- |
| appKey [String]      | yes          | Applications secret key | zHTqS70ca9d3e988946f19a65a01dRR5e56460 |

**Request path parameter:**

| **Parameter [Type]** | **Required** | **Description**                                                                   | **Example**                            |
| -------------------  | ------------ | --------------------------------------------------------------------------------- | -------------------------------------- |
| accessToken          | yes          | token granted for the partner for the respective user number that initiated login | "71d8367e-39f7-4de5-a3a3-2066431b9ca8" |


**Response Codes**

- 200 OK - **Access Token is valid**
- 404 Not Found - **If your credentials are not valid**

    {
       "code": 404,
       "message": "Invalid partner credentials."
    }
- 404 Not Found - **If access token is not valid**

    {
       "code": 1404,
       "message": "Invalid access token."
    }
- 500 Internal Error - **If any other internal error**

    {
       "code": 500,
       "message": "Fail to validate the token"
    }


IMPORTANT: Truecaller SDK already verifies the authenticity of the response before forwarding it to the your app.

#### C. Request-Response correlation check

Every request sent via a Truecaller app that supports truecaller SDK 0.7 has a unique identifier. This identifier is bundled into the response for assuring a correlation between a request and a response. If you want you can check this correlation yourself by:

1. You can use your own custom request identifier via the TrueClient with `TrueSDK.getInstance().setRequestNonce(customHash);`
2. In `ITrueCallback.onSuccesProfileShared(TrueProfile)` verify that the previously generated identifier matches the one in TrueProfile.requestNonce.

IMPORTANT: Truecaller SDK already verifies the Request-Response correlation before forwarding it to the your app.
