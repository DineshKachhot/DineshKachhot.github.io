---
title: Enable Password Autofill (React Native Expo)
layout: post
category: ReactNative
---

Password autofill is an import feature for good user experience, as user will not have to enter username and password every time they login. As a human tendancy to forgot password, they need to reset password everytime they forgot their password, that inconvinance could lead us to loose users.

> From iOS 11 iOS introduce a new feature of password autofill. Password autofill simplifies login and account creation for iOS apps and webpages. With just a few taps, your users can create and save new passwords or log in to an existing account. Users don’t even need to know their password; the system handles everything. This convenience increases the likelihood that users will complete your app’s onboarding process and start using your app more quickly. Additionally, by encouraging users to select unique, strong passwords, you increase the security of your app.

By default, Password AutoFill saves the user’s login credentials on their current iOS device. iOS can sync these credentials securely across the user’s devices using iCloud Keychain. Password AutoFill recommends credentials only for your app’s associated domain, and the user must authenticate using Face ID or Touch ID before accessing these credentials. [Here](https://developer.apple.com/videos/play/wwdc2017/206/) is the WWDC video for password autofill. [Here](https://ali-akhtar.medium.com/all-about-ios-12-autofill-passwords-tool-apis-8f095127fd99) is an detailed tutorial about Password autofill.

> Password strength is a measure of the effectiveness of a password against guessing or brute-force attacks. In its usual form, it estimates how many trials an attacker who does not have direct access to the password would need, on average, to guess it correctly. The strength of a password is a function of length, complexity, and unpredictability.

#### Steps to enable Password AutoFill?
1. Add the Associated Domains Entitlement
2. Add the Apple App Site Association File to your website
3. Set the correct AutoFill type on relevant text fields

### 1. Add the Associated Domains Entitlement

> Associated domains establish a secure association between domains and your app so you can share credentials or provide features in your app from your website. Shared web credentials, universal links, Handoff, and App Clips all use associated domains. 

1. Add associatedDomain config to your app.json/app.config.js 

```javascript
"ios": {
  "bundleIdentifier": "com.myproduct.myproduct-app",
  "associatedDomains": ["webcredentials:www.myproduct.com"]
},
```

Now, when I uploaded a new IPA to AppStore Connect with the new config at this point, I got an error:
> ERROR ITMS-90046: "Invalid Code Signing Entitlements. Your application bundle's signature contains code signing entitlements that are not supported on iOS. Specifically, value '*' for key 'com.apple.developer.associated-domains' in 'Payload/Exponent.app/Exponent' is not supported."

2. Enable Associated Domains entitlement to your app identifier

Open [Developer Apple portal](https://developer.apple.com/account/ios/identifier/bundle) > goto identifier > select your app identifier > select > Capabilities > check Associated Domains

3. Re generate your provisioning profiles

As we added associate domain we have to delete existing provisioning profile and create new provisioning profile.

Open [Developer Apple portal](https://developer.apple.com/account/ios/identifier/bundle) > Profiles > Select your profile > Remove the profile

Run following commands in root folder of your project to create profiles using eas (if you are using expo you can refer expo document)

```terminal
eas credentials

// Select iOS
// Login to your apple account
// Select Team
// Select Provider 
// Select First option 'Build Credentials: Manage everything needed to build your project'
```

eas is awasom!! 😎

As we changed the identity and profile, we have to re-create the application build to reflect those changes.

### 2. Add the Associated Domain File to Your Website

> When a user installs your app, the system attempts to download the associated domain file and verify the domains in your entitlement.

To add the associated domain file to your website, create a file named apple-app-site-association (without an extension).

Update the JSON code in the file for the to add webcredentials service in the file. Be carefull while you are updating the file because it's most error pron step of the process of enable password autofill.

The following JSON code represents the contents of a simple association file.

```json
{
   "webcredentials": {
      "apps": [ "ABCDE12345.com.myproduct.com" ]
   },
}
```

The formation of the item in array is in following formate,

```<Application Identifier Prefix>.<Bundle Identifier>```

After you construct the association file, place it in your site’s .well-known directory. The file’s URL should match the following format:

```https://<fully qualified domain>/.well-known/apple-app-site-association```

So in out case it should be, ```https://myproduct/.well-known/apple-app-site-association```

> You must host the file using https:// with a valid certificate and with no redirects.

### 3. Set the correct AutoFill type on relevant text fields

> For iOS 11+ you can set textContentType to username or password to enable autofill of login details from the device keychain.
> For iOS 12+ newPassword can be used to indicate a new password input the user may want to save in the keychain, and oneTimeCode can be used to indicate that a field can be autofilled by a code arriving in an SMS.
> To disable autofill, set textContentType to none.

Set textContentType prop to text input, for email/username TextInput set textContentType={'username'} and for password set it to 'password'. 

Protip: If you want the update the password in keychain once you update the password in react native app, you have to add email/username TextInput in the screen, but if don't want the TextInput to visible in screen you can set poition={'absolute} in style prop. This is required because we have to give a referance which password entry to update.

#### Conclusion
Password autofill is a good feature to add in app to have a seamless user experience.