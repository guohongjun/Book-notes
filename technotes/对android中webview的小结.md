# android webview的理解

### 1 android webview的基本设置

渲染速度慢

前端H5页面渲染的速度取决于 两个方面：

- Js 解析效率
Js 本身的解析过程复杂、解析速度不快 & 前端页面涉及较多 JS 代码文件，所以叠加起来会导致 Js 解析效率非常低
- 手机硬件设备的性能
由于Android机型碎片化，这导致手机硬件设备的性能不可控，而大多数的Android手机硬件设备无法达到很好很好的硬件性能
总结：上述两个原因 导致 H5页面的渲染速度慢。


## 2 webview 与native交互

- url 截串
- jsbridge
- 读取 cookie


### user-agent说明
为了便于WEB端统计分析，需要将APP的 user-agent 作特征标记，所以搜索了一下Android对webview的User-Agent设置方法，具体如下：

``` java  view plain copy 
// 修改ua使得web端正确判断  
String ua = webview.getSettings().getUserAgentString();  
webview.getSettings().setUserAgentString(ua+"; 自定义标记"); 
``` 

为了便于WEB端统计分析，需要将APP的 user-agent 作特征标记，所以搜索了一下android对webview的User-Agent设置方法，具体如下：



// 修改ua使得web端正确判断
String ua = webview.getSettings().getUserAgentString();
webview.getSettings().setUserAgentString(ua+"; HFWSH /"+appversion);
这种方式是尾部添加的，也可以采用替换的方式



// 修改ua使得web端正确判断

``` java
String ua = webView.getSettings().getUserAgentString();
webView.getSettings().setUserAgentString(ua.replace("Android", "HFWSH_USER Android"));

```

## 通过 WebViewClient 的方法shouldOverrideUrlLoading ()回调拦截 url

基本原理：

- Android通过 WebViewClient 的回调方法shouldOverrideUrlLoading ()拦截 url
- 解析该 url 的协议

## jsbridge
> 在Android中，JSBridge已经不是什么新鲜的事物了，各家的实现方式也略有差异。大多数人都知道WebView存在一个漏洞，见WebView中接口隐患与手机挂马利用，虽然该漏洞已经在Android 4.2上修复了，即使用@JavascriptInterface代替addJavascriptInterface，但是由于兼容性和安全性问题，基本上我们不会再利用Android系统为我们提供的addJavascriptInterface方法或者@JavascriptInterface注解来实现，所以我们只能另辟蹊径，去寻找既安全，又能实现兼容Android各个版本的方案。

jsbridge调用：

- 1 原始jsbridge
- 2 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调拦截JS对话框alert()、confirm()、prompt（） 消息



## 对cookie的操作 


### cookie的定义
> -  Cookie是客户端存储的一种身份凭证，由服务端在回应的消息头中通过Set-Cookie字段“种”在客户端。以后每次客户端在向服务端请求时都会在消息头中带上Cookie字段。服务端就会根据Cookie的来判断此次请求是从哪个用户发过来的，是否是一次有效请求等。
> - Cookie是key-value形式的。每条Cookie都有一个效信息字段，有些还含有expires、domain和path等字段。其中的domain和path字段，区分了不同的cookie可以被哪些网页链接获得.
> - 它会在本地维护每次会话的cookie(保存在data/data/package_name/app_WebView/Cookies.db)。 当WebView加载URL的时候,WebView会从本地读取该URL对应的cookie，并携带该cookie与服务器进行通信。 
WebView通过android.webkit.CookieManager类来维护cookie。CookieManager是WebView的cookie管理类。

### 1 cookie读取

``` java

 /**
     * 读写cookie
     *
     * @param url
     */
    private void readCookies(String url) {
        try {
            CookieManager cookieManager = CookieManager.getInstance();
            // cookie的domain写入的时候的域名
            String cookieStr = cookieManager.getCookie("domain");

            tv.setText(cookieStr);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
     
```

### 2 cookie的写入
``` java

   /**
     * 读写cookie
     *
     * @param url
     * @param expiresDate
     * @param ticket
     */
    private void addCookies(String url, String expiresDate, String ticket) {
        try {
            CookieManager cookieManager = CookieManager.getInstance();

            cookieManager.setAcceptCookie(true);
            // 设置跨域cookie读取
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                CookieManager.getInstance().setAcceptThirdPartyCookies(mWebView, true);
            }
            Calendar calendar = Calendar.getInstance();
            calendar.add(Calendar.WEEK_OF_MONTH, +1);
            cookieManager.setCookie(url, "sa_auth=" + ticket + ";" + "expires=" + calendar.getTime().toString() + ";" + "domain=" + ".elong.com" + ";" + "Path=/" + ";");
            if (Build.VERSION.SDK_INT < 21) {
CookieSyncManager.getInstance().sync();
            } else {
               CookieManager.getInstance().flush();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

```

### 查看cookie工具
- chrome://inspect
- sqlite database browser




 