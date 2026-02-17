# WebView Interactive Components - Integration Guide

## 📋 Overview

This WebView provides interactive overlay components for live streaming applications. It includes brand follow cards, product cards, explore buttons, and reward badges that can be controlled dynamically through `postMessage` communication.

## 🎨 Available Components

### 1. Brand Follow Card
A card displaying brand information with a follow button.

**Features:**
- Brand logo
- Brand name
- Verification badge
- Follower count
- Follow button

### 2. Item Card (Product Card)
A product display card with pricing and purchase option.

**Features:**
- Product image
- Badge number
- Original price (strikethrough)
- Current price
- Discount badge
- "Buy Now" CTA button

### 3. Explore Button
A navigation button (always visible).

**Features:**
- House icon
- "Explorar" text
- Arrow indicator

### 4. Reward Badge
A points/rewards indicator.

**Features:**
- Star icon
- Points display (e.g., "+ 20")

---

## 🔄 Communication Protocol

The WebView uses `postMessage` for bidirectional communication between native Android code and the WebView.

### Incoming Events (Native → WebView)

#### 1. Show Component
Shows a component with optional duration and data.

```json
{
  "type": "showComponent",
  "data": {
    "component": "brandFollowCard",
    "duration": 10000,
    "data": {
      "brandName": "Nike",
      "followers": "+25 mil seguidores",
      "logoUrl": "https://example.com/logo.png",
      "isVerified": true
    }
  }
}
```

**Parameters:**
- `component`: Component name (`brandFollowCard`, `itemCard`, `rewardBadge`)
- `duration`: Duration in milliseconds (0 = infinite)
- `data`: Component-specific data (optional)

#### 2. Hide Component
Hides a component immediately.

```json
{
  "type": "hideComponent",
  "data": {
    "component": "brandFollowCard"
  }
}
```

#### 3. Update Component Data
Updates component data without showing/hiding.

```json
{
  "type": "updateComponentData",
  "data": {
    "component": "itemCard",
    "data": {
      "currentPrice": "$ 750.000",
      "discount": "15% OFF"
    }
  }
}
```

### Outgoing Events (WebView → Native)

#### 1. Follow Button Clicked
```json
{
  "type": "followButtonClicked",
  "data": {
    "brandName": "Adidas"
  }
}
```

#### 2. Buy Now Button Clicked
```json
{
  "type": "buyNowButtonClicked",
  "data": {
    "productId": "3"
  }
}
```

#### 3. Explore Button Clicked
```json
{
  "type": "exploreButtonClicked",
  "data": {}
}
```

#### 4. Reward Badge Clicked
```json
{
  "type": "rewardBadgeClicked",
  "data": {
    "points": "+ 20"
  }
}
```

#### 5. Component Shown
```json
{
  "type": "componentShown",
  "data": {
    "component": "brandFollowCard"
  }
}
```

#### 6. Component Hidden
```json
{
  "type": "componentHidden",
  "data": {
    "component": "brandFollowCard"
  }
}
```

#### 7. WebView Ready
```json
{
  "type": "webViewReady",
  "data": {}
}
```

---

## 💾 Component Data Schemas

### Brand Follow Card Data
```kotlin
data class BrandFollowCardData(
    val brandName: String,
    val followers: String,
    val logoUrl: String? = null,
    val isVerified: Boolean = true
)
```

### Item Card Data
```kotlin
data class ItemCardData(
    val badge: String,
    val imageUrl: String,
    val originalPrice: String,
    val currentPrice: String,
    val discount: String,
    val ctaText: String = "Comprar ahora"
)
```

### Reward Badge Data
```kotlin
data class RewardBadgeData(
    val points: String
)
```

---

## 🔧 Kotlin Integration

### Step 1: WebView Setup

```kotlin
import android.webkit.WebView
import android.webkit.WebChromeClient
import android.webkit.WebViewClient
import android.webkit.JavascriptInterface
import org.json.JSONObject

class InteractiveWebView(context: Context) {
    
    private val webView: WebView = WebView(context).apply {
        settings.apply {
            javaScriptEnabled = true
            domStorageEnabled = true
            allowFileAccess = false
            allowContentAccess = false
        }
        
        // Make background transparent
        setBackgroundColor(Color.TRANSPARENT)
        
        webViewClient = WebViewClient()
        webChromeClient = WebChromeClient()
        
        // Add JavaScript interface
        addJavascriptInterface(WebViewBridge(), "Android")
    }
    
    fun loadWebView() {
        // Load your HTML file
        webView.loadUrl("file:///android_asset/transparent_test.html")
    }
    
    fun getWebView(): WebView = webView
}
```

### Step 2: JavaScript Bridge

```kotlin
class WebViewBridge {
    
    @JavascriptInterface
    fun postMessage(message: String) {
        try {
            val json = JSONObject(message)
            val type = json.getString("type")
            val data = json.optJSONObject("data")
            
            when (type) {
                "followButtonClicked" -> handleFollowClick(data)
                "buyNowButtonClicked" -> handleBuyNowClick(data)
                "exploreButtonClicked" -> handleExploreClick()
                "rewardBadgeClicked" -> handleRewardClick(data)
                "componentShown" -> handleComponentShown(data)
                "componentHidden" -> handleComponentHidden(data)
                "webViewReady" -> handleWebViewReady()
            }
        } catch (e: Exception) {
            Log.e("WebViewBridge", "Error parsing message: $message", e)
        }
    }
    
    private fun handleFollowClick(data: JSONObject?) {
        val brandName = data?.optString("brandName") ?: ""
        // Handle follow action
        Log.d("WebViewBridge", "Follow clicked: $brandName")
    }
    
    private fun handleBuyNowClick(data: JSONObject?) {
        val productId = data?.optString("productId") ?: ""
        // Handle buy now action
        Log.d("WebViewBridge", "Buy now clicked: $productId")
    }
    
    private fun handleExploreClick() {
        // Handle explore action
        Log.d("WebViewBridge", "Explore clicked")
    }
    
    private fun handleRewardClick(data: JSONObject?) {
        val points = data?.optString("points") ?: ""
        // Handle reward click
        Log.d("WebViewBridge", "Reward clicked: $points")
    }
    
    private fun handleComponentShown(data: JSONObject?) {
        val component = data?.optString("component") ?: ""
        Log.d("WebViewBridge", "Component shown: $component")
    }
    
    private fun handleComponentHidden(data: JSONObject?) {
        val component = data?.optString("component") ?: ""
        Log.d("WebViewBridge", "Component hidden: $component")
    }
    
    private fun handleWebViewReady() {
        Log.d("WebViewBridge", "WebView ready")
        // WebView is ready to receive commands
    }
}
```

### Step 3: Sending Messages to WebView

```kotlin
class WebViewController(private val webView: WebView) {
    
    fun showBrandCard(data: BrandFollowCardData, duration: Int = 10000) {
        val message = JSONObject().apply {
            put("type", "showComponent")
            put("data", JSONObject().apply {
                put("component", "brandFollowCard")
                put("duration", duration)
                put("data", JSONObject().apply {
                    put("brandName", data.brandName)
                    put("followers", data.followers)
                    data.logoUrl?.let { put("logoUrl", it) }
                    put("isVerified", data.isVerified)
                })
            })
        }
        sendMessage(message)
    }
    
    fun showItemCard(data: ItemCardData, duration: Int = 10000) {
        val message = JSONObject().apply {
            put("type", "showComponent")
            put("data", JSONObject().apply {
                put("component", "itemCard")
                put("duration", duration)
                put("data", JSONObject().apply {
                    put("badge", data.badge)
                    put("imageUrl", data.imageUrl)
                    put("originalPrice", data.originalPrice)
                    put("currentPrice", data.currentPrice)
                    put("discount", data.discount)
                    put("ctaText", data.ctaText)
                })
            })
        }
        sendMessage(message)
    }
    
    fun showRewardBadge(data: RewardBadgeData, duration: Int = 10000) {
        val message = JSONObject().apply {
            put("type", "showComponent")
            put("data", JSONObject().apply {
                put("component", "rewardBadge")
                put("duration", duration)
                put("data", JSONObject().apply {
                    put("points", data.points)
                })
            })
        }
        sendMessage(message)
    }
    
    fun hideComponent(component: String) {
        val message = JSONObject().apply {
            put("type", "hideComponent")
            put("data", JSONObject().apply {
                put("component", component)
            })
        }
        sendMessage(message)
    }
    
    fun updateComponentData(component: String, data: JSONObject) {
        val message = JSONObject().apply {
            put("type", "updateComponentData")
            put("data", JSONObject().apply {
                put("component", component)
                put("data", data)
            })
        }
        sendMessage(message)
    }
    
    private fun sendMessage(message: JSONObject) {
        webView.post {
            val script = "window.postMessage(${message}, '*');"
            webView.evaluateJavascript(script, null)
        }
    }
}
```

### Step 4: Usage Example in Activity/Fragment

```kotlin
class LiveStreamActivity : AppCompatActivity() {
    
    private lateinit var webViewController: WebViewController
    private lateinit var interactiveWebView: InteractiveWebView
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Initialize WebView
        interactiveWebView = InteractiveWebView(this)
        webViewController = WebViewController(interactiveWebView.getWebView())
        
        // Add to layout
        val container = findViewById<FrameLayout>(R.id.webview_container)
        container.addView(interactiveWebView.getWebView())
        
        // Load WebView
        interactiveWebView.loadWebView()
        
        // Example: Show brand card after 2 seconds
        Handler(Looper.getMainLooper()).postDelayed({
            showBrandExample()
        }, 2000)
    }
    
    private fun showBrandExample() {
        val brandData = BrandFollowCardData(
            brandName = "Nike",
            followers = "+25 mil seguidores",
            logoUrl = "https://example.com/nike-logo.png",
            isVerified = true
        )
        webViewController.showBrandCard(brandData, duration = 10000)
    }
    
    private fun showProductExample() {
        val productData = ItemCardData(
            badge = "1",
            imageUrl = "https://example.com/product.jpg",
            originalPrice = "$ 150.000",
            currentPrice = "$ 99.000",
            discount = "34% OFF",
            ctaText = "Comprar ahora"
        )
        webViewController.showItemCard(productData, duration = 10000)
    }
    
    private fun showRewardExample() {
        val rewardData = RewardBadgeData(points = "+ 50")
        webViewController.showRewardBadge(rewardData, duration = 10000)
    }
}
```

### Step 5: Layout XML

```xml
<FrameLayout
    android:id="@+id/webview_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/transparent" />
```

---

## 🎯 Best Practices

### 1. **Lifecycle Management**
```kotlin
override fun onPause() {
    super.onPause()
    webView.onPause()
}

override fun onResume() {
    super.onResume()
    webView.onResume()
}

override fun onDestroy() {
    super.onDestroy()
    webView.destroy()
}
```

### 2. **Memory Management**
```kotlin
// Clear WebView cache when needed
webView.clearCache(true)
webView.clearHistory()
```

### 3. **Error Handling**
```kotlin
webView.webViewClient = object : WebViewClient() {
    override fun onReceivedError(
        view: WebView?,
        request: WebResourceRequest?,
        error: WebResourceError?
    ) {
        Log.e("WebView", "Error loading: ${error?.description}")
    }
}
```

### 4. **Threading**
Always post WebView operations on the main thread:
```kotlin
runOnUiThread {
    webViewController.showBrandCard(data)
}
```

---

## 🎨 Animations

All components include smooth slide-in/slide-out animations:
- **Brand Card & Item Card**: Slide from left
- **Reward Badge**: Slide from right
- **Duration**: 0.5s (in), 0.4s (out)
- **Easing**: cubic-bezier(0.4, 0, 0.2, 1)

---

## 🐛 Debug Mode

The WebView includes a debug panel (bottom of screen) with buttons to test all components. To hide it in production, add this JavaScript:

```kotlin
webView.evaluateJavascript("debugTogglePanel()", null)
```

Or remove the debug panel from the HTML before deployment.

---

## 📱 Responsive Design

All components are optimized for mobile devices with:
- Compact sizes (180px wide item cards)
- Small font sizes (10-14px)
- Touch-friendly buttons (min 44dp)
- Proper spacing for thumbs

---

## 🔒 Security Considerations

1. **Disable unnecessary WebView features:**
```kotlin
settings.apply {
    allowFileAccess = false
    allowContentAccess = false
    setGeolocationEnabled(false)
}
```

2. **Validate all incoming messages:**
```kotlin
@JavascriptInterface
fun postMessage(message: String) {
    if (!isValidMessage(message)) {
        return
    }
    // Process message
}
```

3. **Use HTTPS for remote content:**
```kotlin
if (url.startsWith("http://")) {
    // Block or convert to HTTPS
}
```

---

## 📚 Additional Resources

- [WebView Documentation](https://developer.android.com/reference/android/webkit/WebView)
- [postMessage API](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)
- [JavaScript Interface](https://developer.android.com/guide/webapps/webview#BindingJavaScript)

---

## 🆘 Troubleshooting

### Issue: Components not showing
- Check if JavaScript is enabled
- Verify the HTML file is loaded correctly
- Check console logs for errors

### Issue: postMessage not working
- Ensure JavascriptInterface is added
- Check if Android object is available in WebView
- Verify JSON format

### Issue: Animations not smooth
- Enable hardware acceleration:
```kotlin
webView.setLayerType(View.LAYER_TYPE_HARDWARE, null)
```

### Issue: Transparent background not working
- Set background color to transparent:
```kotlin
webView.setBackgroundColor(Color.TRANSPARENT)
```

---

## 📄 License.

**Created with 💕**
