# Enliq Linkify for Android

Dynamic links, deferred deep links and App Links for Android apps, by Enliq. Works with Google Play and Huawei AppGallery.

The latest version is **1.1.0**. It needs Android 5.0 (API 21) or newer.

## Install

The SDK is served from this repository's Maven repository, with no token. Add it next to your other repositories:

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        exclusiveContent {
            forRepository { maven("https://loodos.github.io/enliq-linkify-android-sdk") }
            filter { includeModule("com.loodos.enliq", "linkify") }
        }
    }
}
```

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.loodos.enliq:linkify:1.1.0")
}
```

## Set up

Add your Enliq hosts inside `<application>` in `AndroidManifest.xml`:

```xml
<meta-data android:name="eiq.host" android:value="https://api.example.com" />
<meta-data android:name="eiq.domain" android:value="https://link.example.com" />
<meta-data android:name="eiq.appIndexUrl" android:value="https://www.example.com" />
```

- `eiq.host`: the Enliq API.
- `eiq.domain`: your dynamic link domain.
- `eiq.appIndexUrl`: your website, for App Links.

Initialize the SDK in your `Application`:

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        EIQLinkify.initialize(context = this, loggingEnabled = BuildConfig.DEBUG)
    }
}
```

## Handle links

Pass every link that opens your activity to `processUrl`. It resolves dynamic links and App Links into your app's deep link, and the first launch after an install into the link that led to it:

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        handleLink(intent)
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        handleLink(intent)
    }

    private fun handleLink(intent: Intent?) {
        val url = intent?.dataString ?: return
        EIQLinkify.processUrl(this, url)
            .addOnSuccessListener { deepLink -> /* open deepLink */ }
            .addOnFailureListener { error -> /* not an Enliq link, or it could not be resolved */ }
    }
}
```

`handleDynamicLink` and `handleAppIndexingURL` handle only one kind of link, when you need that.

## Basket events

After the app was opened through a Linkify link, report basket changes and orders:

```kotlin
EIQLinkify.basketEvent(
    context = applicationContext,
    basketId = "basket-789",
    totalAmount = 131.97,
    status = BasketEventType.ADD_TO_BASKET, // or ORDER_SUCCESS, REMOVE_FROM_BASKET
    products = listOf(
        BasketProduct(id = "product-123", name = "Headphones", quantity = 1, price = 99.99, totalAmount = 99.99),
        BasketProduct(id = "product-456", name = "Phone case", quantity = 2, price = 15.99, totalAmount = 31.98),
    ),
)
```

The link's key is kept in memory until the app process ends; `clearSession()` drops it sooner.

## Releases

See this repository's [releases](../../releases) for each version.
