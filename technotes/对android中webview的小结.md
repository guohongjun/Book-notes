#android webview的理解

### 1 android webview的基本设置



## 2 webview 与native交互

- url 截串
- jsbridge
- 读取 cookie



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


 