---
# required metadata

title: Microsoft Intune App SDK for iOS developer guide | Microsoft Docs
description: The Microsoft Intune App SDK for iOS lets you incorporate Intune app protection policies--in the form of mobile app management (MAM)--into your iOS app.
keywords:
author: mtillman
manager: angrobe
ms.author: mtillman
ms.date: 12/15/2016
ms.topic: article
ms.prod:
ms.service: microsoft-intune
ms.technology:
ms.assetid: 8e280d23-2a25-4a84-9bcb-210b30c63c0b

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: oydang
ms.suite: ems
#ms.tgt_pltfrm:
#ms.custom:

---

# Microsoft Intune App SDK for iOS developer guide

> [!NOTE]
> You might want to first read the [Get Started with Intune App SDK Guide](intune-app-sdk-get-started.md) article, which explains how to prepare for integration on each supported platform.

The Microsoft Intune App SDK for iOS lets you incorporate Intune app protection policies--in the form of mobile app management (MAM)--into your iOS app. A MAM-enabled application is one that's integrated with the Intune App SDK. It lets IT admins deploy policies to your mobile app when Intune actively manages the app.

## Prerequisites

* You will need a Mac OS computer that runs OS X 10.8.5 or later and has the Xcode toolset version 5 or later installed.

* Review the [Intune App SDK for iOS License Terms](https://github.com/msintuneappsdk/ms-intune-app-sdk-ios/blob/master/Microsoft%20License%20Terms%20Intune%20App%20SDK%20for%20iOS%20.pdf). Print and retain a copy of the license terms for your records. By downloading and using the Intune App SDK for iOS, you agree to such license terms.  If you do not accept them, do not use the software.

* Download the files for the Intune App SDK for iOS on [GitHub](https://github.com/msintuneappsdk/ms-intune-app-sdk-ios).

## What’s in the SDK

The Intune App SDK for iOS includes a static library, resource files, API headers, a debug settings .plist file, and a configurator tool. Mobile apps might simply include the resource files and statically link to the libraries for most policy enforcement. Advanced Intune MAM features are enforced through APIs.

This guide covers the use of the following components of the Intune App SDK for iOS:

* **libIntuneMAM.a**: The Intune App SDK static library. If your app does not use extensions, link this library to your project to enable your app for Intune mobile application management.

* **IntuneMAM.framework**: The Intune App SDK framework. Link this framework to your project to enable your app for Intune mobile application management. Use the framework instead of the static library if your app uses extensions, so that your project does not create multiple copies of the static library.

* **IntuneMAMResources.bundle**: A resource bundle that has resources that the SDK relies on.

* **Headers**: Exposes the Intune App SDK APIs. If you use an API, you will need to include the header file that contains the API. The following header files include the API function calls required to enable the functionality of the Intune App SDK:

	* IntuneMAMAsyncResult.h
	* IntuneMAMDataProtectionInfo.h
	* IntuneMAMDataProtectionManager.h
	* IntuneMAMFileProtectionInfo.h
	* IntuneMAMFileProtectionManager.h
	* IntuneMAMPolicyDelegate.h
	* IntuneMAMLogger.h


## How the Intune App SDK works

The objective of the Intune App SDK for iOS is to add management capabilities to iOS applications with minimal code changes. The fewer the code changes, the less time to market--without affecting the consistency and stability of your mobile application.

The application needs to be linked to the static library and must include the resource bundle. The MAMDebugSettings.plist file is optional. It can be included in the package to simulate MAM policies being applied to the application without requiring you to deploy the application via Microsoft Intune. Additionally, in debug builds, you can apply the policies in the MAMDebugSettings.plist file by transferring the file to the app’s Documents directory via iTunes file sharing.

## Build the SDK into your mobile app

To enable the Intune App SDK, follow these steps:

1. **Option 1**: Link to the `libIntuneMAM.a` library. Drag the `libIntuneMAM.a` library to the **Linked Frameworks and Libraries** list of the project target.
    ![Intune App SDK iOS: linked frameworks and libraries](../media/intune-app-sdk-ios-linked-frameworks-and-libraries.png)

	> [!NOTE]
	> If you plan to release your app to the App Store, please use the version of `libIntuneMAM.a` that is built for release and not the debug version. The release version will be in the **release** folder. The debug version has verbose output that helps troubleshoot problems with the Intune App SDK.

    **Option 2**: Link `IntuneMAM.framework` to your project. Drag `IntuneMAM.framework` to the **Linked Frameworks and Libraries** list of the project target.

	> [!NOTE]
	> If you use the framework, you must manually strip out the simulator architectures from the universal framework before you submit your app to the App Store. See the section "Submitting your app to the App Store."

2. Add these iOS frameworks to the project:
    * MessageUI.framework
    * Security.framework
    * MobileCoreServices.framework
    * SystemConfiguration.framework
    * libsqlite3.dylib
    * libc++.dylib
    * ImageIO.framework
    * LocalAuthentication.framework
    * AudioToolbox.framework

	> [!NOTE]
	> If the application is targeted for iOS 7, set the `Status` attribute of `LocalAuthentication.framework` to Optional. If `Status` is not set, the application will fail to start on iOS 7.
	>
	> Also, Xcode 7 has switched `.dylib` extensions to `.tbd`.

3. Add the `IntuneMAMResources.bundle` resource bundle to the project by dragging the resource bundle under **Copy Bundle Resources** within **Build Phases**.
![Intune App SDK iOS: copy bundle resources](../media/intune-app-sdk-ios-copy-bundle-resources.png)

4. Add `-force_load {PATH_TO_LIB}/libIntuneMAM.a` to either of the following, replacing `{PATH_TO_LIB}` with the Intune App SDK location:
    * The project’s `OTHER_LDFLAGS` build configuration setting
    * The UI’s **Other Linker Flags**<br>

	> [!NOTE]
	> To find `PATH_TO_LIB`, select the file `libIntuneMAM.a` and choose **Get Info** from the **File** menu. Copy and paste the **Where** information (the path) from the **General** section of the **Info** window.

5. If your mobile app defines a main nib or storyboard file in its Info.plist file, remove the **Main Storyboard** or **Main Nib** field. Add the storyboard or nib values you removed previously under a new dictionary named IntuneMAMSettings with the following key names, as applicable:
    * MainStoryboardFile
    * MainStoryboardFile~ipad
    * MainNibFile
    * MainNibFile~ipad

	> [!NOTE]
    > If your mobile app doesn’t define a main nib or storyboard file in its Info.plist file, these settings are not required.

	You can view Info.plist in raw format (to see the key names) by right-clicking anywhere in the document body and changing the view type to **Show Raw Keys/Values**.

6. Enable keychain sharing (if it isn't already enabled) by choosing **Capabilities** in each project target and enabling the **Keychain Sharing** switch. Keychain sharing is required for you to proceed to the next step.

    > [!NOTE]
	> Your provisioning profile needs to support new keychain sharing values. The keychain access groups should support a wildcard character. You can check this by opening the .mobileprovision file in a text editor, searching for **keychain-access-groups**, and ensuring that you have a wildcard. For example:
	```xml
	<key>keychain-access-groups</key>
	<array>
	<string>YOURBUNDLESEEDID.*</string>
	</array>
	```

7. After you enable keychain sharing, follow these steps to create a separate access group in which the Intune App SDK data will be stored. You can create a keychain access group by using the UI or by using the entitlements file.

    If you're using the UI to create the keychain access group:

    a. If your mobile app does not have any keychain access groups defined, add the app’s bundle ID as the first group.

    b. Add the shared keychain group `com.microsoft.intune.mam`. The Intune App SDK uses this access group to store data.

    c. Add `com.microsoft.adalcache` to your existing access groups.

	![Intune App SDK iOS: keychain sharing](../media/intune-app-sdk-ios-keychain-sharing.png)

    If you're using the entitlement file to create the keychain access group, prepend the keychain access group with `$(AppIdentifierPrefix)` in the entitlement file. For example:  

    * `$(AppIdentifierPrefix)com.microsoft.intune.mam`
	* `$(AppIdentifierPrefix)com.microsoft.adalcache`

	> [!NOTE]
	> An entitlements file is an XML file that's unique to your mobile application. It's used to specify special permissions and capabilities in your iOS app.

8. If the app defines URL schemes in its Info.plist file, add another scheme, with a `-intunemam` suffix, for each URL scheme.

9. For mobile apps developed for iOS 9+, include each protocol that your app passes to `UIApplication canOpenURL` in the `LSApplicationQueriesSchemes` array of your app's Info.plist file. Additionally, for each protocol listed, add a new protocol and append it with `-intunemam`. You must also include `http-intunemam`, `https-intunemam`, and `ms-outlook-intunemam` in the array.

10. If the app has app groups defined in its entitlements, add these groups to the IntuneMAMSettings dictionary under the `AppGroupIdentifiers` key as an array of strings.

11. Link your mobile application to the Azure Directory Authentication Library (ADAL). The ADAL library for Objective C is [available on GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc).

    > [!NOTE]
	> The Intune App SDK has been tested against the ADAL broker branch code from June 19, 2015. Please ensure that you are linking with the latest/working version of the ADAL library.

12. Include the `ADALiOSBundle.bundle` resource bundle in the project by dragging the resource bundle under **Copy Bundle Resources** within **Build Phases**.

13. Use the `-force_load PATH_TO_ADAL_LIBRARY` linker option when you're linking to the library.

    Add `-force_load {PATH_TO_LIB}/libADALiOS.a` to the project’s `OTHER_LDFLAGS` build configuration setting or **Other Linker Flags** in the UI. `PATH_TO_LIB` should be replaced with the location of the ADAL binaries.

## Set up Azure Directory Authentication Library

The Intune App SDK uses ADAL for its authentication and conditional launch scenarios. It also relies on ADAL to register the user identity with the MAM service for management without device enrollment scenarios.

Typically, ADAL requires apps to register with Azure Active Directory (Azure AD) and get a unique ID (known as the client ID), and other identifiers, to guarantee the security of the tokens granted to the app. The Intune App SDK uses default registration values when it contacts Azure AD.  

If the app itself uses ADAL for its authentication scenario, the app must use its existing registration values and override the Intune App SDK default values. This ensures that users are not prompted for authentication twice (once by the Intune App SDK and once by the app).

### ADAL FAQs

**What ADAL binaries should I use?**

The Intune App SDK currently uses the broker branch of [ADAL on GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc) to support apps that require conditional access. (These apps therefore depend on the Microsoft Authenticator app.) But the SDK is still compatible with the master branch of ADAL. Use the branch that is appropriate for your app.

**How do I link to ADAL binaries?**

Add `-force_load {PATH_TO_LIB}/libADALiOS.a` to the project’s `OTHER_LDFLAGS` build configuration setting or **Other Linker Flags** in the UI. `PATH_TO_LIB` should be replaced with the location of the ADAL binaries. Also, make sure to copy the ADAL bundle to your app.  

For more details, see the instructions from [ADAL on GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc).

**How do I share the ADAL cache with other apps signed with the same provisioning profile?**

If your app does not have any keychain access groups defined, add the app’s bundle ID as the first group.

Enable ADAL single sign-on (SSO) by adding `com.microsoft.adalcache` and `com.microsoft.workplacejoin` access groups in the keychain entitlements.

If you are explicitly setting the ADAL shared cache keychain group, make sure it is set to `<app_id_prefix>.com.microsoft.adalcache`. ADAL will set this for you unless you override it. If you want to specify a custom keychain group to replace `com.microsoft.adalcache`, specify that in the Info.plist file under IntuneMAMSettings, by using the key `ADALCacheKeychainGroupOverride`.

**How do I force the Intune App SDK to use ADAL settings that my app already uses?**

If your app already uses ADAL, see the section on IntuneMAMSettings for information on populating the following settings:  

* ADALClientId
* ADALRedirectUri
* ADALRedirectScheme
* ADALCacheKeychainGroupOverride

**How do I switch between Azure AD production and internal test environments?**

You can use the `AadAuthorityURI` setting in MAMPolicies.plist to specify the Azure AD environment used for ADAL calls. It’s currently set to use the Azure AD preproduction environment (PPE) by default unless it's overridden.

To test against PPE, you can use a compile-time or runtime switch.

For a compile-time environment switch of MAM service URLs and Azure AD, set the `UsePPE` Boolean flag to true in MAMEnvironment.plist. (Note that there is no support for doing this via Info.plist.)

For a runtime environment switch, set `com.microsoft.intune.mam.useppe` in standard user defaults to “1” to use PPE. This replaces the existing `com.microsoft.intune.mam.AADAuthorityEnvironment` setting.

**How do I override the Azure AD authority URL with a tenant-specific URL supplied at runtime?**

Set the `aadAuthorityUriOverride` property on the IntuneMAMPolicyManager instance.

> [!NOTE]
> You would need this in the scenario of MAM without device enrollment to let the SDK reuse the ADAL refresh token fetched by the app.

The SDK will continue to use this authority URL for policy refresh and any subsequent enrollment requests, unless the value is cleared or changed.  So it's important to clear the value when a corporate user signs out of the app and reset it when a new corporate user signs in.

**What should I do if my app itself uses ADAL for authentication?**

The following actions are required if the app already uses ADAL for authentication:

* In the project’s Info.plist file, under the IntuneMAMSettings dictionary with the key name `ADALClientId`, specify the client ID to be used for ADAL calls.

* In the project’s Info.plist file, under the IntuneMAMSettings dictionary with the key name `ADALRedirectUri`, specify the redirect URI to be used for ADAL calls. You might also need to specify `ADALRedirectScheme`, depending on the format of your app’s redirect URI.

**What if my app does not already use ADAL for authentication?**

If your app does not use ADAL, the Intune App SDK will provide default values for ADAL parameters and handle authentication against Azure AD.

## Register your app with the Intune MAM service

### Use the APIs
The Intune App SDK now provides the ability for iOS apps to receive MAM policies from Intune without the need to be enrolled with Intune through mobile device management (MDM). To support this new functionality, the SDK provides new APIs that let the app receive MAM policies. To use the new APIs, follow these steps:

1. Use the latest release of the Intune App SDK, which supports management of apps with or without device enrollment. If your app has used an older version of the SDK without this feature, you will need to update the Intune MAM library, as well as update the Headers folder with the headers from the latest SDK.

2. Add IntuneMAMEnrollment.h to any files that will call the APIs.

3. To test against PPE, you can use a compile-time or runtime switch.

	For a compile-time environment switch of MAM service URLs and Azure AD, set the `UsePPE` Boolean flag to true in MAMEnvironment.plist. (Note that there is no support for doing this via Info.plist.)

	For a runtime environment switch, set `com.microsoft.intune.mam.useppe` in standard user defaults to “1” to use PPE. This replaces the existing `com.microsoft.intune.mam.AADAuthorityEnvironment` setting.


### Register accounts

An app can receive MAM policies from the Intune service if the app is enrolled on behalf of a specified user account. The app is responsible for registering any newly signed-in user with the Intune App SDK. After the new user account has been authenticated, the app should call the `registerAndEnrollAccount` method in Headers/IntuneMAMEnrollment.h:

```objc
/**


 *  This method will add the account to the list of registered accounts.
 *  An enrollment request will immediately be started.
 *  @param identity The UPN of the account to be registered with the SDK
 */

(void)registerAndEnrollAccount:(NSString *)identity;

```
By calling the `registerAndEnrollAccount` method, the SDK will register the user account and attempt to enroll the app on behalf of this account. If the enrollment fails for any reason, the SDK will automatically retry the enrollment 24 hours later. For debugging purposes, the app can receive notifications, via a delegate, about the results of any enrollment requests.

After this API has been invoked, the application can continue to function as normal. If the enrollment succeeds, the SDK will notify the user that an app restart is required. At that time, the user can immediately restart the app.

### Deregister accounts

Before a user is signed out of an app, the app should deregister the user from the SDK. This will ensure:

1. Enrollment retries will no longer happen for the user’s account.

2. If the user has successfully enrolled the application, the user and app will be unenrolled from the Intune MAM service, and MAM policies will be removed.

3. If the app initiates a selective wipe (optional), any work-related or school-related data is deleted.

Before the user is signed out, the app should call the following API in Headers/IntuneMAMEnrollment.h:

```objc
/*
 *  This method will remove the provided account from the list of
 *  registered accounts.  Once removed, if the account has enrolled
 *  the application, the account will be un-enrolled.
 *  @note In the case where an un-enroll is required, this method will block
 *  until the Intune MAM AAD token is acquired, then return.  This method must be called before  
 *  the user is removed from the application (so that required AAD tokens are not purged
 *  before this method is called).
 *  @param identity The UPN of the account to be removed.
 *  @param doWipe   If YES, a selective wipe if the account is un-enrolled
 */

(void)deRegisterAndUnenrollAccount:(NSString *)identity withWipe:(BOOL)doWipe;
```

This method must be called before the user account’s Azure AD tokens are deleted. The SDK needs the user’s app token to make specific requests to the Intune MAM service on behalf of the user.

If the app will delete the user’s work-related or school-related data on its own, the `doWipe` flag can be set to false. Otherwise, the app can have the SDK initiate a selective wipe. This will result in a call to the app's selective wipe delegate.

```objc
[[IntuneMAMEnrollmentManager instance] deRegisterAndUnenrollAccount:@”user@foo.com” withWipe:YES];
```

## Enroll without prior sign-in

An app that does not sign in the user with Azure Active Directory can still receive MAM policies from the Intune service by calling the API to have the SDK handle that authentication. Apps should use this technique when they have not authenticated a user with Azure AD but still need to retrieve MAM policies to help protect data. An example is if another authentication service is being used for app sign-in, or if the app does not support signing in at all. To do this, the application should call the
`loginAndEnrollAccount`  method in Headers/IntuneMAMEnrollment.h:

```objc
/**
 *  Creates an enrollment request which is started immediately.
 *  If no token can be retrieved for the identity, the user will be prompted
 *  to enter their credentials, after which enrollment will be retried.
 *  @param identity The UPN of the account to be logged in and enrolled.
 */
 (void)loginAndEnrollAccount: (NSString *)identity;

```

By calling this method, the SDK will prompt the user for credentials if no existing token can be found. The SDK will then try to enroll the application on behalf of this account. The method can be called with "nil" as the identity. In that case, the SDK will enroll with the existing MAM user on the device, or prompt the user for a user name if no existing user is found.

If the enrollment fails, the app should consider calling this API again at a future time, depending on the details of the failure. The app can receive notifications, via a delegate, about the results of any enrollment requests.

After this API has been invoked, the app can continue functioning as normal. If the enrollment succeeds, the SDK will notify the user that an app restart is required.

## Status, result, and debug notifications

The app can receive status, result, and debug notifications about the following requests to the Intune MAM service:

 - Enrollment requests
 - Policy update requests
 - Unenrollment requests

The notifications are presented via delegate methods in Headers/IntuneMAMEnrollmentDelegate.h:

```objc
/**
 *  Called when an enrollment request operation is completed.
 * @param status status object containing debug information
 */

(void)enrollmentRequestWithStatus:(IntuneMAMEnrollmentStatus *)status;

/**
 *  Called when a MAM policy request operation is completed.
 *  @param status status object containing debug information
 */
(void)policyRequestWithStatus:(IntuneMAMEnrollmentStatus *)status;

/**
 *  Called when a un-enroll request operation is completed.
 *  @Note: when a user is un-enrolled, the user is also de-registered with the SDK
 *  @param status status object containing debug information
 */

(void)unenrollRequestWithStatus:(IntuneMAMEnrollmentStatus *)status;

```

These delegate methods return an `IntuneMAMEnrollmentStatus` object that has the following information:

- The identity of the account associated with the request
- A status code that indicates the result of the request
- An error string with a description of the status code
- An `NSError` object

This object is defined in Headers/IntuneMAMEnrollmentStatus.h, along with the specific status codes that can be returned.




## Sample code

These are example implementations of the delegate methods:

```objc
- (void)enrollmentRequestWithStatus:(IntuneMAMEnrollmentStatus *)status


{


    NSLog(@"enrollment result for identity %@ with status code %ld", status.identity, (unsigned long)status.statusCode);


    NSLog(@"Debug Message: %@", status.errorString);


}


- (void)policyRequestWithStatus:(IntuneMAMEnrollmentStatus *)status


{


    NSLog(@"policy check-in result for identity %@ with status code %ld", status.identity, (unsigned long)status.statusCode);


    NSLog(@"Debug Message: %@", status.errorString);


}


- (void)unenrollRequestWithStatus:(IntuneMAMEnrollmentStatus *)status


{


    NSLog(@"un-enroll result for identity %@ with status code %ld", status.identity, (unsigned long)status.statusCode);



    NSLog(@"Debug Message: %@", status.errorString);


}

```

## App restart

When an app receives MAM policies for the first time, it must restart to apply the required hooks. To notify the app that a restart needs to happen, the SDK provides a delegate method in Headers/IntuneMAMPolicyDelegate.h.

```objc
 - (BOOL) restartApplication
```
The return value of this method tells the SDK if the application will handle the required restart:   

 - If true is returned, the application will handle the restart.   
 - If false is returned, the SDK will restart the application after this method returns. The SDK will immediately show a dialog box that tells the user to restart the application.

## Implement save-as controls

Intune lets IT admins select which storage locations a managed app can save data to. Apps can query the Intune App SDK for allowed storage locations by using the **isSaveToAllowedForLocation** API.

Before apps can save managed data to a cloud-storage or local location, they must check with the **isSaveToAllowedForLocation** API to know if the IT admin has allowed data to be saved there.

When apps use the **isSaveToAllowedForLocation** API, they must pass in the UPN for the storage location, if it is available.

### Supported save locations

The **isSaveToAllowedForLocation** API provides constants to check whether the IT admin permits data to be saved to the following locations:

* IntuneMAMSaveLocationOther
* IntuneMAMSaveLocationOneDriveForBusiness
* IntuneMAMSaveLocationSharePoint
* IntuneMAMSaveLocationBox
* IntuneMAMSaveLocationDropbox
* IntuneMAMSaveLocationGoogleDrive
* IntuneMAMSaveLocationLocalDrive

Apps should use the constants in the **isSaveToAllowedForLocation** API to check if data can be saved to locations considered "managed," like OneDrive for Business, or "personal." Additionally, the API should be used when the app can't check whether a location is "managed" or "personal."

When a location is known to be "personal," apps should use the **IntuneMAMSaveLocationOther** value.

The **IntuneMAMSaveLocationLocalDrive** constant should be used when the app is saving data to any location on the local device.

## Set up the Intune App SDK

You use the IntuneMAMSettings dictionary in the application’s Info.plist file to set up the Intune App SDK. The following table lists all supported settings.

Some of these settings might have been covered in previous sections, and some do not apply to all apps.

Setting  | Type  | Definition | Required?
--       |  --   |   --       |  --
ADALClientId  | String  | The app’s Azure AD client identifier. | Required if the app uses ADAL.
ADALRedirectUri  | String  | The app’s Azure AD redirect URI. | ADALRedirectUri or ADALRedirectScheme is required if the app uses ADAL.
ADALRedirectScheme  | String  | The app's Azure AD redirect scheme. This can be used in place of ADALRedirectUri if the application's redirect URI is in the format `scheme://bundle_id`. | ADALRedirectUri or ADALRedirectScheme is required if the app uses ADAL.
ADALLogOverrideDisabled | Boolean  | Specifies whether the SDK will route all ADAL logs (including ADAL calls from the app, if any) to its own log file. Defaults to NO. Set to YES if the app will set its own ADAL log callback. | Optional.
ADALCacheKeychainGroupOverride | String  | Specifies the keychain group to use for the ADAL cache, instead of “com.microsoft.adalcache." Note that this doesn’t have the app-id prefix. That will be prefixed to the provided string at runtime. | Optional.
AppGroupIdentifiers | Array of string  | Array of app groups from the app’s entitlements com.apple.security.application-groups section. | Required if the app uses application groups.
ContainingAppBundleId | String | Specifies the bundle ID of the extension’s containing application. | Required for iOS extensions.
DebugSettingsEnabled| Boolean | If set to YES, test policies within the Settings bundle can be applied. Applications should *not* be shipped with this setting enabled. | Optional.
MainNibFile<br>MainNibFile~ipad  | String  | This setting should have the application’s main nib file name.  | Required if the application defines MainNibFile in Info.plist.
MainStoryboardFile<br>MainStoryboardFile~ipad  | String  | This setting should have the application’s main storyboard file name. | Required if the application defines UIMainStoryboardFile in Info.plist.
MAMPolicyRequired| Boolean| Specifies whether the app will be blocked from starting if the app does not have an Intune MAM policy. Defaults to NO. | Optional.
MAMPolicyWarnAbsent | Boolean| Specifies whether the app will warn the user during launch if the app does not have an Intune MAM policy. Note that apps cannot be submitted to the store with this setting set to YES. | Optional.
MultiIdentity | Boolean| Specifies whether the app is multi-identity aware. | Optional.
SplashIconFile <br>SplashIconFile~ipad | String  | Specifies the Intune splash (startup) icon file. | Optional.
SplashDuration | Number | Minimum amount of time, in seconds, that the Intune startup screen will be shown at application launch. Defaults to 1.5. | Optional.
BackgroundColor| String| Specifies the background color for the startup and PIN screens. Accepts a hexadecimal RGB string in the form of #XXXXXX, where X can range from 0-9 or A-F. The pound sign might be omitted.   | Optional. Defaults to light grey.
ForegroundColor| String| Specifies the foreground color for the startup and PIN screens, like text color. Accepts a hexadecimal RGB string in the form of #XXXXXX, where X can range from 0-9 or A-F. The pound sign might be omitted.  | Optional. Defaults to black.
AccentColor | String| Specifies the accent color for the PIN screen, like button text color and box highlight color. Accepts a hexadecimal RGB string in the form of #XXXXXX, where X can range from 0-9 or A-F. The pound sign might be omitted.| Optional. Defaults to system blue.
MAMTelemetryDisabled| Boolean| Specifies if the SDK will not send any telemetry data to its back end.| Optional.
MAMTelemetryUsePPE | Boolean | Specifies if the SDK will send data to the PPE back end. Use this when testing your apps with an Intune policy so that test telemetry data does not mix with customer data. | Optional.

## Telemetry

By default, the Intune App SDK for iOS logs telemetry data on the following usage events. This data is sent to Microsoft Intune.

* **App launch**: To help Microsoft Intune learn about MAM-enabled app usage by management type (MAM with MDM, MAM without MDM enrollment, and so on).
* **EnrollApplication API call**: To help Microsoft Intune learn about success rate and other performance metrics of `enrollApplication` calls from the client side.

> [!NOTE]
> If you choose not to send Intune App SDK telemetry data to Microsoft Intune from your mobile application, you must disable Intune App SDK telemetry capture. Set the property `MAMTelemetryDisabled` to YES in the IntuneMAMSettings dictionary.

## Enable multi-identity (optional)

By default, the SDK applies a policy to the app as a whole. Multi-identity is a MAM feature that you can enable to apply a policy on a per-identity level. This requires more app participation than other MAM features.

The app must inform the app SDK when it intends to change the active identity. The SDK also notifies the app when an identity change is required. Currently, only one managed identity is supported. After the user enrolls the device or the app, the SDK uses this identity and considers it the primary managed identity. Other users in the app will be treated as unmanaged with unrestricted policy settings.

Note that an identity is simply defined as a string. Identities are case-insensitive. Requests to the SDK for an identity might not return the same casing that was originally used when the identity was set.

### Identity overview

An identity is simply the user name of an account (for example, user@contoso.com). Developers can set the identity of the app on the following levels:

* **Process identity**: Sets the process-wide identity and is mainly used for single identity applications. This identity affects all tasks, files, and UI.
* **UI identity**: Determines what policies are applied to UI tasks on the main thread, like cut/copy/paste, PIN, authentication, and data sharing. The UI identity does not affect file tasks like encryption and backup.
* **Thread identity**: Affects what policies are applied on the current thread. This identity affects all tasks, files, and UI.

The app is responsible for setting the identities appropriately, whether or not the user is managed.

At any time, every thread has an effective identity for UI tasks and file tasks. This is the identity that's used to check what policies, if any, should be applied. If the identity is "no identity" or the user is not managed, no policies will be applied.

### Thread queues

Apps often dispatch asynchronous and synchronous tasks to thread queues. The SDK intercepts Grand Central Dispatch (GCD) calls and associates the current thread identity with the dispatched tasks. When the tasks are finished, the SDK temporarily changes the thread identity to the identity associated with the tasks, finishes the tasks, then restores the original thread identity.


Because `NSOperationQueue` is built on top of GCD, `NSOperations` will run on the identity of the thread at the time the tasks are added to `NSOperationQueue`. `NSOperations` or functions dispatched directly through GCD can also change the current thread identity as they are running. This identity will override the identity inherited from the dispatching thread.

### File owner

The SDK tracks the identities of local file owners and applies policies accordingly. A file owner is established when a file is created or when a file is opened in truncate mode. The owner is set to the effective file task identity of the thread that's performing the task.

Alternatively, apps can set the file owner identity explicitly by using `IntuneMAMFilePolicyManager`. Apps can use `IntuneMAMFilePolicyManager` to retrieve the file owner and set the UI identity before showing the file contents.

### Shared data

If the app creates files that have data from both managed and unmanaged users, the app is responsible for encrypting the managed user’s data. You can encrypt data by using the `protect` and `unprotect` APIs in `IntuneMAMDataProtectionManager`.

The `protect` method accepts an identity that can be a managed or unmanaged user. If the user is managed, the data will be encrypted. If the user is unmanaged, a header will be added to the data that's encoding the identity, but the data will not be encrypted. You can use the `protectionInfo` method to retrieve the data’s owner.

### Share extensions

If the app has a share extension, the owner of the item being shared can be retrieved through the  `protectionInfoForItemProvider` method in `IntuneMAMDataProtectionManager`. If the shared item is a file, the SDK will handle setting the file owner. If the shared item is data, the app is responsible for setting the file owner if this data is persisted to a file, and for calling the `setUIPolicyIdentity` API before showing this data in the UI.

### Turning on multi-identity

By default, apps are considered single identity. The SDK sets the process identity to the enrolled user. To enable multi-identity support, add a Boolean setting with the name `MultiIdentity` and a value of YES to the IntuneMAMSettings dictionary in the app's Info.plist file.

> [!NOTE]
> When multi-identity is enabled, the process identity, UI identity, and thread identities are set to nil. The app is responsible for setting them appropriately.

### Switching identities

* **App-initiated identity switch**:

	At launch, multi-identity apps are considered to be running under an unknown, unmanaged account. The conditional launch UI will not run, and no policies will be enforced on the app. The app is responsible for notifying the SDK whenever the identity should be changed. Typically, this will happen whenever the app is about to show data for a specific user account.

	An example is when the user attempts to open a document, a mailbox, or a tab in a notebook. The app needs to notify the SDK before the file, mailbox, or tab is actually opened. This is done through the `setUIPolicyIdentity` API in `IntuneMAMPolicyManager`. This API should be called whether or not the user is managed. If the user is managed, the SDK will perform the conditional launch checks, like jailbreak detection, PIN, and authentication.

	The result of the identity switch is returned to the app asynchronously through a completion handler. The app should postpone opening the document, mailbox, or tab until a success result code is returned. If the identity switch failed, the app should cancel the task.

* **SDK-initiated identity switch**:

	Sometimes, the SDK needs to ask the app to switch to a specific identity. Multi-identity apps must implement the `identitySwitchRequired` method in `IntuneMAMPolicyDelegate` to handle this request.

	When this method is called, if the app can handle the request to switch to the specified identity, it should pass `IntuneMAMAddIdentityResultSuccess` into the completion handler. If it can't handle switching the identity, the app should pass `IntuneMAMAddIdentityResultFailed` into the completion handler.

	The app does not have to call `setUIPolicyIdentity` in response to this call. If the SDK needs the app to switch to an unmanaged user account, the empty string will be passed into the `identitySwitchRequired` call.

* **Selective wipe**:

	When the app is selectively wiped, the SDK will call the `wipeDataForAccount` method in `IntuneMAMPolicyDelegate`. The app is responsible for removing the specified user’s account and any data associated with it. The SDK is capable of removing all files owned by the user and will do so if the app returns FALSE from the `wipeDataForAccount` call.

	Note that this method is called from a background thread. The app should not return a value until all data for the user has been removed (with the exception of files if the app returns FALSE).

## Debug the Intune App SDK in Xcode

Before you manually test your MAM-enabled app with Microsoft Intune, you can use a Settings.bundle file while in Xcode. This will let you set test policies without requiring a connection to Intune. To enable it:

1. Add a Settings.bundle file by right-clicking the top-level folder in your project. Choose **Add** > **New File** from the menu. Under **Resources**, choose the **Settings Bundle** template to add.

2. On debug builds only, copy MAMDebugSettings.plist into Settings.bundle.

3. In Root.plist (which is in Settings.bundle), add a preference with `Type` = `Child Pane` and `FileName` = `MAMDebugSettings`.

4. In **Settings** > **Your-App-Name**, turn on **Enable Test Policies**.

5. Start the app (either inside or outside Xcode).

6. In **Settings** > **Your-App-Name** > **Enable Test Policies**, turn on a policy--for example, **PIN**.

7. Start the app (either inside or outside Xcode). Check that the PIN works as expected.

> [!NOTE]
> You can use **Settings** > **Your-App-Name** > **Enable Test Policies** to enable and switch settings.

## iOS best practices

Here are recommended best practices for developing for iOS:

* The iOS file system is case-sensitive. Ensure that the case is correct for file names like `libIntuneMAM.a` and `IntuneMAMResources.bundle`.

* If Xcode has trouble finding `libIntuneMAM.a`, you can fix the problem by adding the path to this library into the linker search paths.

## FAQ


**Are all of the APIs addressable through native Swift or the Objective-C and Swift interoperability?**

The Intune App SDK APIs are in objective-C only and do not support native Swift.  


**Do all users of my application need to be registered with the MAM service?**

No. In fact, only work or school accounts should be registered with the Intune App SDK. Apps are responsible for determining if an account is used in a work or school context.   

**What about users that have already signed in to the application? Do they need to be enrolled?**

The application is responsible for enrolling users after they have been successfully authenticated. The application is also responsible for enrolling any existing accounts that might have been present before the application had MDM-less MAM functionality.   

To do this, the application should make use of the `registeredAccounts:` method. This method returns an NSDictionary that has all of the accounts registered into the Intune MAM service. If any existing accounts in the application are not in the list, the application should register and enroll those accounts via `registerAndEnrollAccount:`.

**How often does the SDK retry enrollments?**

The SDK will automatically retry all previously failed enrollments on a 24-hour interval. The SDK does this to ensure that if a user’s organization enabled MAM after the user signed in to the application, the user will successfully enroll and receive policies.

The SDK will stop retrying when it detects that a user has successfully enrolled the application. This is because only one user can enroll an application at a particular time. If the user is unenrolled, the retries will begin again on the same 24-hour interval.

**Why does the user need to be deregistered?**

The SDK will take these actions in the background periodically:

 - If the application is not yet enrolled, it will try to enroll all registered accounts every 24 hours.
 - If the application is enrolled, the SDK will check for MAM policy updates every 8 hours.

Deregistering a user notifies the SDK that the user will no longer use the application, and the SDK can stop any of the periodic events for that user account. It also triggers an app unenroll and selective wipe if necessary.

**Should I set the doWipe flag to true in the deregister method?**

This method should be called before the user is signed out of the application.  If the user’s data is deleted from the application as part of the sign-out, `doWipe` can be set to false. But if the application does not remove the user’s data, `doWipe` should be set to true so that the SDK can delete the data.

**Are there any other ways that an application can be un-enrolled?**

Yes, the IT admin can send a selective wipe command to the application. This will deregister and unenroll the user, and it will wipe the user’s data. The SDK automatically handles this scenario and sends a notification via the unenroll delegate method.



## Submit your app to the App Store

Both the static library and framework builds of the Intune App SDK are universal binaries. This means they have code for all device and simulator architectures. Apple will reject apps submitted to the App Store if they have simulator code. When compiling against the static library for device-only builds, the linker will automatically strip out the simulator code. Follow the steps below to ensure all simulator code is removed before you upload your app to the App Store.

1. Make sure `IntuneMAM.framework` is on your desktop.

2. Run these commands:

	```bash
	lipo ~/Desktop/IntuneMAM.framework/IntuneMAM -remove i386 -remove x86_64 -output ~/Desktop/IntuneMAM.device_only
	```

	```bash
	cp ~/Desktop/IntuneMAM.device_only ~/Desktop/IntuneMAM.framework/IntuneMAM
	```
	The first command strips the simulator architectures from the framework's DYLIB file. The second command copies the device-only DYLIB file back into the framework directory.
