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

In first step, you need to call `Zowie.initialize()` method in your Application's onCreate() method.
(Don't forget to register your application class in AndroidManifest.xml)

```kotlin
class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        Zowie.initialize(application = this)
    }
}
```

In second step, you have to create `ZowieConfiguration` and set it by calling `Zowie.setConfiguration()`

```kotlin
val configuration = ZowieConfiguration(
    instanceId = "YOUR_INSTANCE_ID",
    chatHost = "YOUR_CHAT_HOST", // e.g. "yourbrand.chat.getzowie.com/api/v1"
    authenticationType = ZowieAuthenticationType.Anonymous, // Or ZowieAuthenticationType.JwtToken(...)

    // OPTIONAL SETTINGS
    // Optional: Start a specific flow by referral key
    conversationInitReferral = "OPTIONAL_CONVERSATION_INIT_REFERRAL",

    // Optional: Show welcome message when chat opens for the first time (default = true)
    startOnOpen = true
)

Zowie.setConfiguration(configuration)
```

If your integration requires token authentication you can replace `ZowieAuthenticationType.Anonymous` with `ZowieAuthenticationType.JwtToken(userId, conversationId, token)`.

`ZowieConfiguration` supports these optional values:

- `conversationInitReferral`: starts a specific flow by referral key.
- `description`: overrides chat description.
- `fontColor`: overrides configured font color with `ZowieFontColor.WHITE` or `ZowieFontColor.BLACK`.
- `logoUrl`: overrides chat logo.
- `primaryColor`: overrides configured primary color. Use a valid Android color string, e.g. `#FF9900`.
- `userMessageBackgroundColor`: overrides app-user message bubble color. Use a valid Android color string.
- `userMessageFontColor`: overrides app-user message font color with `ZowieFontColor.WHITE` or `ZowieFontColor.BLACK`.
- `startOnOpen`: controls whether the welcome message starts when chat opens for the first time.
- `title`: overrides chat title.
- `voiceBlobColor`: overrides voice blob color.
- `voiceExperienceEnabled`: enables or disables voice entry points locally.
- `initialConversationMode`: controls the first screen with `ZowieConversationMode.TEXT` or `ZowieConversationMode.VOICE`. Voice mode is used only when voice experience is enabled locally and remotely.

### Chat UI

Zowie chat is a simple android Fragment.
An instance can be obtained by calling `Zowie.createChatFragment()`.
Then it can be shown for example using fragment transaction.

```kotlin
val chatFragment = Zowie.createChatFragment() ?: return
supportFragmentManager.beginTransaction().apply {
    replace(R.id.your_fragment_container, chatFragment)
    commitAllowingStateLoss()
}
```

If the chat is embedded as a Fragment, implement `ZowieOnChatClosedListener`
on the host `Activity` or parent `Fragment` to control what happens after the user ends the chat.

```kotlin
class ChatActivity : AppCompatActivity(), ZowieOnChatClosedListener {

    override fun onChatClosed() {
        // Navigate away from the chat, e.g. finish this Activity or pop your navigation stack.
        finish()
    }
}
```

You can also launch chat as a dedicated sdk `Activity`.

```kotlin
Zowie.openChat(context = this)
```

If you want to control activity launch flags or use Activity Result API, create an intent first:

```kotlin
val intent = Zowie.createChatIntent(context = this) ?: return
startActivity(intent)
```

## Full-screen chat (optional)

### What “full-screen” means

**Android can draw under the status bar / navigation bar only when the host Activity is in edge-to-edge mode**  
(i.e. the window does *not* fit system windows / the app draws behind system bars).

If your Activity is **NOT** edge-to-edge, the system already restricts the app content area below system bars,
so the chat **cannot** render behind them — no flag in the SDK can change that without modifying the host window.

### Enable / disable in SDK

- `false` (default): chat UI is laid out *below* the status bar / cutout and *above* the navigation bar, with IME handling.
- `true`: chat UI draws behind status & navigation bars **(requires edge-to-edge on the host Activity)**. IME padding remains so the input stays above the keyboard.

```kotlin
Zowie.setChatFullScreenEnabled(true)
```

**Important:** Set the flag **before creating** the chat UI (before adding the fragment). Otherwise you may see a short “jump”.

```kotlin
Zowie.setChatFullScreenEnabled(true)

val chatFragment = Zowie.createChatFragment() ?: return
supportFragmentManager.beginTransaction().apply {
    replace(R.id.your_fragment_container, chatFragment)
    commitAllowingStateLoss()
}
```

### Edge-to-edge requirement

To get true full-screen (draw behind system bars), your Activity must be edge-to-edge.

Typical setup:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    enableEdgeToEdge() // or WindowCompat.setDecorFitsSystemWindows(window, false)
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_chat)

    Zowie.setChatFullScreenEnabled(true)

    val chatFragment = Zowie.createChatFragment() ?: return
    supportFragmentManager.beginTransaction()
        .replace(R.id.your_fragment_container, chatFragment)
        .commitAllowingStateLoss()
}
```

### Does SDK force edge-to-edge?

**No.** The SDK does **not** change your Activity window flags automatically.  
Forcing edge-to-edge from inside an SDK can break the host app’s layout, so it must be controlled by the app.

If your Activity is not edge-to-edge, setting `Zowie.setChatFullScreenEnabled(true)` will **not** make the chat draw under system bars.

## Window Insets & Edge-to-Edge

Our SDK automatically handles system window insets (status bar, navigation bar, display cutouts, and on-screen keyboard) to ensure a correct layout.

If your app applies global window inset handling at the Activity or root container level (e.g., using `ViewCompat.setOnApplyWindowInsetsListener`),
make sure you don’t apply the same insets again to the container that hosts the SDK fragment (avoid “double padding”).

If needed, consume the insets at your root level:

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(root) { _, _ ->
    // Handle your app's views here

    // Prevent double insets in nested content (including SDK fragment)
    WindowInsetsCompat.CONSUMED
}
```

Or exclude the chat container from global inset handling.

## Chat customization

### Layout configuration

Use `Zowie.setLayoutConfiguration()` to control consultant avatar and consultant name display.

```kotlin
val layoutConfiguration = ZowieLayoutConfiguration {
    showConsultantAvatar = true
    consultantNameMode = ZowieConsultantNameMode.FIRST_NAME
}

Zowie.setLayoutConfiguration(layoutConfiguration)
```

Available `ZowieConsultantNameMode` values are:

- `HIDDEN`
- `FIRST_NAME`
- `FULL_NAME`

### File upload configuration

Use `Zowie.setFileUploadConfiguration()` to enable file upload and restrict allowed file types.

```kotlin
val fileUploadConfiguration = ZowieFileUploadConfiguration {
    allowFileUpload = true
    allowFileTypes = listOf(
        ZowieFileType.JPEG,
        ZowieFileType.PNG,
        ZowieFileType.GIF,
        ZowieFileType.MP4,
        ZowieFileType.PDF
    )
}

Zowie.setFileUploadConfiguration(fileUploadConfiguration)
```

Supported `ZowieFileType` values are: `JPEG`, `PNG`, `GIF`, `MP4`, `PDF`, `MP3`, `AAC`, `DOC`, `DOCX`, `TXT`, `ZIP`, `RAR`, `ZIP7`, `HTML`, `SPX`, `MOV`, `AVI`, `XLS`, `XLSX`, and `CSV`.

## Setting user metadata

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

## FCM notifications

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

## Context

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

## Anonymous session

To clear Anonymous session call `Zowie.clearAnonymousSession()`

```kotlin
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

## Initialization error

To set initialization error listener call `Zowie.setOnInitializationErrorListener()`

```kotlin
Zowie.setOnInitializationErrorListener {
    // initialization error action
}
```

## Setting URL handler

By default, Zowie Sdk opens URLs using external web browser. If you want to customize this behaviour use `Zowie.setUrlHandler()`

```kotlin
val zowieUrlHandler = object : ZowieUrlHandler {
    override fun onUrl(url: String, source: ZowieUrlActionSource): Boolean {
        return if (YOUR_CONDITION) {
            // custom url handling
            true // block default sdk behaviour
        } else {
            // use default sdk behaviour
            false
        }
    }
}
Zowie.setUrlHandler(zowieUrlHandler)
```

## Custom events

You can listen for backend events with with `Zowie.on(eventName, handler)`.

- `eventName` must match backend event name exactly.
- `params` is delivered as `Any?`:
    - JSON object -> `Map<String, Any?>`
    - JSON array -> `List<Any?>`
    - plain value / invalid JSON -> raw `String`
    - missing value -> `null`

```kotlin
Zowie.on("meetingDetails") { params ->
    // handle event payload
}
```

To remove a handler for a given event, pass `null`:

```kotlin
Zowie.on("meetingDetails", null)
```

## AI Session Notice "More" handler

By default, tapping the "More" link on the AI session notice opens a built-in bottom sheet showing the full message. You can override this behaviour (in both text and voice chat) by setting your own handler with `Zowie.setAiSessionNoticeMoreHandler()`. The handler receives the notice `header` and `message`.

```kotlin
Zowie.setAiSessionNoticeMoreHandler { header, message ->
    // handle the tap yourself, e.g. show your own dialog
}
```

To restore the default bottom sheet, pass `null`:

```kotlin
Zowie.setAiSessionNoticeMoreHandler(null)
```

## Setting Referral value

You can use `setReferral` method before or after chat window is visible.

Setting referral before opening chat window will result in `Waiting` status being emitted by the status listener. This call will wait for chat window to be initialized, and then `referral` will be sent.
If there is no messages in the conversation `referral` value will be used instead of `conversationInitReferral` set in `ZowieConfiguration`.

Setting referral when chat window is already visible and initialized will send a `referral` value immediately.

```kotlin
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

## Setting session timeout

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

## Breaking changes from previous versions

This release changes several public APIs:

- `ZowieConfiguration` is now a Kotlin `data class`. Use the constructor instead of `ZowieConfiguration { ... }` or `ZowieConfiguration.Builder()`.
- `ZowieConfiguration.fontColor` and `ZowieConfiguration.userMessageFontColor` use `ZowieFontColor?` instead of `String?`.
- `Zowie.onVisualAidEvent(eventName, handler)` was renamed to `Zowie.on(eventName, handler)`.
- `Zowie.setChatMode()` and `ZowieChatMode` were removed. Use `voiceExperienceEnabled` and `initialConversationMode` in `ZowieConfiguration`.
- `Zowie.setColors()` and `ZowieColors` were removed. Use remote configuration and the local overrides available in `ZowieConfiguration`.
- `Zowie.setStrings()` and `ZowieStrings` were removed.
- `Zowie.setUserStatus()` was removed. The SDK manages user status internally.
- `Zowie.createChatFragment()` and `Zowie.createChatIntent()` can return `null` when remote configuration cannot be loaded. Handle the nullable result before opening chat.
