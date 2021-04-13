
# Zowie Android SDK

  

## Installation
1. Add Zowie maven repository to your project's `build.gradle` file:
```groovy
allprojects {
    repositories {
        ...
        maven {
            url  "https://zowieteam.bintray.com/zowie-android-sdk"
        }
    }
}
```

2. Add the following dependency to your app's `build.gradle` file:
```groovy
dependencies {
    ...
    implementation 'ai.zowie:android-sdk:0.0.8'
}

```
## Usage

  

### Initialization

Zowie Android SDK initialization is a two step process.

In first step, you need to call `Zowie.initalize()` method in your Application's onCreate() method. (Don't forget to register your application class in AndroidManifest.xml)

```kotlin
class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        Zowie.initialize(application = this)
    }

}
```
\
In second step, you have to create `ZowieConfiguration` and set it by calling `Zowie.setConfiguration()`
```kotlin
val configuration = ZowieConfiguration {
    userId = "YOUR_USER_ID"
    conversationId = "YOUR_CONVERSATION_ID"
    instanceId = "YOUR_INSTANCE_ID"
    conversationInitReferral = "OPTIONAL_CONVERSATION_INIT_REFERRAL"
    authenticationType = ZowieAuthenticationType.Raw 
}

Zowie.setConfiguration(configuration)
```
Alternatively you can use **Builder** to create `ZowieConfiguration`.\
**Builders are available for all classes needed for sdk setup.**
```kotlin
val configuration = 
    ZowieConfiguration.Builder()
        .setUserId("YOUR_USER_ID")
        .setConversationId("YOUR_CONVERSATION_ID")
        .setInstanceId("YOUR_INSTANCE_ID")
        .setConversationInitReferral("OPTIONAL_CONVERSATION_INIT_REFERRAL")
        .setAuthenticationType(ZowieAuthenticationType.Raw)
        .build()
        
Zowie.setConfiguration(configuration)
```
If your integration requires token authentication you can replace `ZowieAuthenticationType.Raw()` with `ZowieAuthenticationType.JwtToken("YOUR_TOKEN")`.


### Chat UI
Zowie chat is a simple android Fragment.
An instance can be obtained by calling `Zowie.createChatFragment()`.
Then it can be shown for example using fragment transaction.
```kotlin
val chatFragment = Zowie.createChatFragment()
supportFragmentManager.beginTransaction().apply {
    replace(R.id.your_fragment_container, chatFragment)
    commitAllowingStateLoss()
}
```
### Setting user metadata

You can set user metadata by calling `Zowie.setMetadata()` method.

```kotlin
val metadata = ZowieMetadata {
    firstName = "FIRST_NAME"
    lastName = "LAST_NAME"
    name = "NAME"
    locale = "LOCALE"
    timeZone = "TIME_ZONE"
    phoneNumber = "PHONE_NUMBER"
    email = "EMAIL"
    extraParams = mapOf(
        "KEY_1" to "VALUE_1",
        "KEY_2" to "VALUE_2"
    )
}

Zowie.setMetadata(
    metadata = metadata,
    onErrorListener = { 
         // on error action 
    },
    onSuccessListener = {
         // on success action
    }
)
```

### FCM notifications
To receive push notifications you have to call `Zowie.enableNotifications()` with Firebase device token.

```kotlin
Zowie.enableNotifications(
    fcmToken = "YOUR_FIREBASE_TOKEN",
    onErrorListener = {
        // on error action
    },
    onSuccessListener = {
        // on success action
    }
)
```
\
If you want to disable notifications call `Zowie.disableNotifications()`
```kotlin
Zowie.disableNotifications(
    onErrorListener = {
        // on error action
    },
    onSuccessListener = {
        // on success action
    }
)
```

###  User status
To set user's status call `Zowie.setUserStatus()`
```kotlin
Zowie.setUserStatus(
    isActive = YOUR_BOOLEAN_VALUE,
    onErrorListener = {
        // on error action
    },
    onSuccessListener = {
        // on success action
    }
)
```

### Context
To set context call `Zowie.setContext()`
```kotlin
Zowie.setContext(
    contextId = "YOUR_CONTEXT",
    onErrorListener = { 
        // on error action 
    },
    onSuccessListener = {
        // on success action
    }
)
```
## Customization

You can fully customize chat colors and strings. **Make sure that customization methods are called before showing chat fragment.**

### Localization

The only supported language in this SDK is `english`. If you need more localization please provide it as below:

```kotlin
val zowieStrings = ZowieStrings {
    newMessageHint = YOUR_VALUE
    messageStatusDelivered = YOUR_VALUE
    messageStatusSendingErrorMessage = YOUR_VALUE
    messageStatusSendingErrorTryAgain = YOUR_VALUE
    readAndWriteStoragePermissionAlertTitle = YOUR_VALUE
    readAndWriteStoragePermissionAlertMessage = YOUR_VALUE
    readAndWriteStoragePermissionAlertPositiveButton = YOUR_VALUE
    readAndWriteStoragePermissionAlertNegativeButton = YOUR_VALUE
    attachmentPlaceholderName = YOUR_VALUE
    attachmentFileMaxSizeExceededErrorMessage = YOUR_VALUE
    couldNotOpenFileErrorMessage = YOUR_VALUE
    chatConnectionErrorMessage = YOUR_VALUE
    chatConnectionRestoredMessage = YOUR_VALUE
    chatHistoryDownloadErrorMessage = YOUR_VALUE
    couldNotOpenWebBrowserErrorMessage = YOUR_VALUE
    unexpectedErrorMessage = YOUR_VALUE
    fileDownloadErrorMessage = YOUR_VALUE
}

Zowie.setStrings(zowieStrings)
```


### Colors
Feel free to set up color branding however you like.

```kotlin
val zowieColors =  ZowieColors {
    messageStatusSendingErrorColor = YOUR_VALUE
    messageStatusDeliveredColor = YOUR_VALUE
    sentMessageBackgroundColor = YOUR_VALUE
    sentMessageContentsColor = YOUR_VALUE
    sentMessageImageUploadLoadingColor = YOUR_VALUE
    sentMessageVideoUploadLoadingColor = YOUR_VALUE
    sentMessageImagePlaceholderLoadingColor = YOUR_VALUE
    sentMessageImagePlaceholderBackgroundColor = YOUR_VALUE
    sentMessageLinksColor = YOUR_VALUE
    incomingMessageBackgroundColor = YOUR_VALUE
    incomingMessagePrimaryTextColor = YOUR_VALUE
    incomingMessageSecondaryTextColor = YOUR_VALUE
    incomingMessageFileIconColor = YOUR_VALUE
    incomingMessageFileDownloadSuccessIconColor = YOUR_VALUE
    incomingMessageDownloadFileIconColor = YOUR_VALUE
    incomingMessageDownloadFileLoadingColor = YOUR_VALUE
    incomingMessageImagePlaceholderLoadingColor = YOUR_VALUE
    incomingMessageImagePlaceholderBackgroundColor = YOUR_VALUE
    incomingMessageLinksColor = YOUR_VALUE
    backgroundColor = YOUR_VALUE
    newMessageTextColor = YOUR_VALUE
    newMessageHintTextColor = YOUR_VALUE
    sendAttachmentButtonColor = YOUR_VALUE
    sendTextButtonColor = YOUR_VALUE
    separatorColor = YOUR_VALUE
    chatMessagesLoadingColor = YOUR_VALUE
    quickButtonBackgroundColor = YOUR_VALUE
    quickButtonBackgroundPressedColor = YOUR_VALUE
    quickButtonTextColor = YOUR_VALUE
    quickButtonPressedStrokeColor = YOUR_VALUE
    quickButtonStrokeColor = YOUR_VALUE
    actionButtonBackgroundColor = YOUR_VALUE
    actionButtonBackgroundPressedColor = YOUR_VALUE
    actionButtonTextColor = YOUR_VALUE
    videoThumbnailPlaceholderColor = YOUR_VALUE
    notificationErrorContentsColor = YOUR_VALUE
    notificationErrorBackgroundColor = YOUR_VALUE
    notificationSuccessContentsColor = YOUR_VALUE
    notificationSuccessBackgroundColor = YOUR_VALUE
    zowieLogoButtonBackgroundPressedColor = YOUR_VALUE
    zowieLogoButtonPressedStrokeColor = YOUR_VALUE
    playVideoButtonBackgroundColor = YOUR_VALUE
    playVideoButtonBackgroundPressedColor = YOUR_VALUE
    playVideoButtonPlayIconColor = YOUR_VALUE
    typingAnimationTintColor = YOUR_VALUE
}

Zowie.setColors(zowieColors)
```
