# Zowie Android SDK

[![Platform](https://img.shields.io/badge/platform-Android-green.svg)](https://github.com/chatbotizeteam/sdk-native-android)
[![jFrog release](https://img.shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Fzowie.jfrog.io%2Fartifactory%2Fzowie-android-sdk%2Fai%2Fzowie%2Fandroid-sdk%2Fmaven-metadata.xml&label=jfrog)](https://zowie.jfrog.io/artifactory/zowie-android-sdk)

## Installation

1. Add Zowie maven repository to your project's `build.gradle` file:

```groovy
allprojects {
    repositories {
        ...
        maven {
            url "https://zowie.jfrog.io/artifactory/zowie-android-sdk"
        }
    }
}
```

2. Add the following dependency to your app's `build.gradle` file:

```groovy
dependencies {
    // ...
    implementation "ai.zowie:android-sdk:$zowieVersion"
}

```

## Usage

### Initialization

Zowie Android SDK initialization is a two step process.

In first step, you need to call `Zowie.initalize()` method in your Application's onCreate()method. (
Don't forget to register your application class in AndroidManifest.xml)

```kotlin
class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        Zowie.initialize(application = this)
    }

}
```

\
In second step, you have to create `ZowieConfiguration` and set it by
calling `Zowie.setConfiguration()`

```kotlin
val configuration = ZowieConfiguration {
    instanceId = "YOUR_INSTANCE_ID"
    chatHost = "YOUR_CHAT_HOST" // e.g. "yourbrand.chat.getzowie.com/api/v1/core"
    authenticationType = ZowieAuthenticationType.Anonymous // Or .JwtToken("userID", "conversationID", "token")


    // OPTIONAL SETTINGS
    // Optional: Start a specific flow by referral key
    conversationInitReferral = "OPTIONAL_CONVERSATION_INIT_REFERRAL"

    // Optional: Show welcome message when chat opens for the first time (default = true)
    startOnOpen = true
}

Zowie.setConfiguration(configuration)
```


Alternatively you can use **Builder** to create `ZowieConfiguration`.\
**Builders are available for all classes needed for sdk setup.**

```kotlin
val configuration =
    ZowieConfiguration.Builder()
        .setInstanceId("YOUR_INSTANCE_ID")
        .setConversationInitReferral("OPTIONAL_CONVERSATION_INIT_REFERRAL")
        .setAuthenticationType(ZowieAuthenticationType.Anonymous)
        .setChatHost("YOUR_CHAT_HOST")
        .build()

Zowie.setConfiguration(configuration)
```

If your integration requires token authentication you can
replace `ZowieAuthenticationType.Anonymous` with `ZowieAuthenticationType.JwtToken()`.

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

To receive push notifications you have to call `Zowie.enableNotifications()` with Firebase device
token.

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

### (DEPRECATED) User status

Setting user status from outside of ZowieSdk is no longer needed an should be removed from client
apps. Current version of ZowieSdk checks, and sets user status internally.

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

### Anonymous session

To clear Anonymous session call `Zowie.clearAnonymousSession()`

```
Zowie.clearAnonymousSession(
    instanceId = "YOUR_INSTANCE_ID",
    onErrorListener = {
        // on error action
    },
    onSuccessListener = {
        // on success action
    }
)
```

### Initialization error

To set initialization error listener call `Zowie.setOnInitializationErrorListener()`

```
Zowie.setOnInitializationErrorListener {
    // initialization error action
}
```

### Setting URL handler

By default, Zowie Sdk opens URLs using external web browser. If you want to customize this behaviour
use `Zowie.setUrlHandler()`

```
val zowieUrlHandler = object : ZowieUrlHandler {
    override fun onUrl(url: String, source: ZowieUrlActionSource): Boolean {
        return if (YOUR_CONDITION) {
            // custom url handling
            return true // block default sdk behaviour
        } else {
            // use default sdk behaviour
            false
        }
    }
}
Zowie.setUrlHandler(zowieUrlHandler)
```

### Setting Referral value

You can use `setReferral` method before or after chat widow is visible.

Setting referral before opening chat window will result in `Waiting` status being emitted by the
status listener. This call will wait for chat window to be initialized, and then `referral` will be sent.
If there is no messages in the conversation `referral` value will be used instead of `conversationInitReferral` set
in `ZowieConfiguration`. 

Setting referral when chat window is already visible and initialized will send a `referral` value
immediately.

```
val listener = ZowieReferralStatusListener{
    when(it){
        is ZowieReferralStatus.Error -> {
            // your code
        }
        ZowieReferralStatus.Success -> {
            // your code
        }
        ZowieReferralStatus.Waiting -> {
            // your code
        }
    }
}
Zowie.setReferral(
    referral = "YOUR_REFERRAL",
    listener = listener
)
```

### Setting session timeout

Sets the duration of user inactivity (in milliseconds) after which the session will automatically timeout.

Parameters:
- `timeout`: `Long`  

The duration (in milliseconds) of inactivity after which the session should be considered expired. 
For example, `300_000` (5 minutes).

- `onSessionTimeout`: `ZowieOnSessionTimeoutListener` 

> Callback invoked when the session expires due to user inactivity.  
> **Implementers must close (remove) the Zowie chat fragment inside this callback.**  
> Once the session times out, the SDK automatically clears all internal session data — **the chat will no longer be functional until a new session is started**.

Usage Example:

```kotlin
Zowie.setSessionTimeout(timeout = 300_000) {
    // Close Zowie SDK on sessionTimeout, e.g. requireActivity().finish() 
}
```

## Customization

You can fully customize chat colors and strings. **Make sure that customization methods are called
before showing chat fragment.**

### Localization

The only supported language in this SDK is `english`. If you need more localization please provide
it as below:

```kotlin
val zowieStrings = ZowieStrings {
    newMessageHint = YOUR_VALUE
    messageStatusDelivered = YOUR_VALUE
    messageStatusRead = YOUR_VALUE
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

Current values:
```
    newMessageHint = "Your message…"
    messageStatusDelivered = "Delivered"
    messageStatusRead = "Read"
    messageStatusSendingErrorMessage = "We couldn't sent your message."
    messageStatusSendingErrorTryAgain = "Try again"
    readAndWriteStoragePermissionAlertTitle = "Permission denied"
    readAndWriteStoragePermissionAlertMessage = "To download files go to system settings and enable storage permission."
    readAndWriteStoragePermissionAlertPositiveButton = "Go to settings"
    readAndWriteStoragePermissionAlertNegativeButton = "No, thanks"
    attachmentPlaceholderName = "Attachment"
    attachmentFileMaxSizeExceededErrorMessage = "Attachment can have 8MB max!"
    couldNotOpenFileErrorMessage = "Could not open file"
    chatConnectionErrorMessage = "We couldn't connect to our servers"
    chatConnectionRestoredMessage = "Connection restored"
    chatHistoryDownloadErrorMessage = "Chat history download error"
    couldNotOpenWebBrowserErrorMessage = "Could not open web browser"
    unexpectedErrorMessage = "Unexpected error"
    fileDownloadErrorMessage = "File download error"
```

### Colors

Feel free to set up color branding however you like.

```kotlin
val zowieColors = ZowieColors {
    messageStatusSendingErrorColor = YOUR_VALUE
    messageStatusDeliveredColor = YOUR_VALUE
    messageStatusReadColor = YOUR_VALUE
    messageAuthorNameTextColor = YOUR_VALUE
    sectionDateTimeTextColor = YOUR_VALUE
    messageDateTimeTextColor = YOUR_VALUE
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
    announcementBackgroundColor = YOUR_VALUE
    announcementStrokeColor = YOUR_VALUE
    announcementTextColor = YOUR_VALUE
    announcementIconColor = YOUR_VALUE
}

Zowie.setColors(zowieColors)
```

### Layout configuration

You can customize chat layout behaviour using `Zowie.setLayoutConfiguration()` method.

```kotlin
val layoutConfiguration = ZowieLayoutConfiguration {
    showConsultantAvatar = YOUR_VALUE
    consultantNameMode = YOUR_VALUE
}
Zowie.setLayoutConfiguration(layoutConfiguration)
```

## Common issues:

### RxJava3 - ApolloNetworkException thrown as UndeliverableException

**This issue may occur in versions from `0.0.1` to `0.0.14`. In version `0.0.15` RxJava3 usage was
removed from ZowieSdk.**
During initialization ZowieSdk sets RxJava3 error handler (`RxJavaPlugins.setErrorHandler`)
internally but only if it's not set by your app.
ZowieSdk also uses `apollo-kotlin` internally, and because of that there is a possibility
that `ApolloNetworkException` will be thrown as RxJava `UndeliverableException`.
If your app sets RxJava error handler, be sure to catch `ApolloNetworkException`, otherwise
unexpected crashes can occur.
If your app does not set error handler, all `UndeliverableExceptions` will be consumed by ZowieSdk.
