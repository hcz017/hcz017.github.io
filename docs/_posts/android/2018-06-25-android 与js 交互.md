---
title: android 与javascript 交互
date: 2018-06-25 21:53
---
# 准备（前提）条件
1. android app 中有使用到webview
2. webview 加载本地或者网络端的html

# 对于Android调用JS代码的方法有：
1. 通过WebView的loadUrl()
2. 通过WebView的evaluateJavascript()
## 代码展示
```java
        // 设置与Js交互的权限
        mWebView.getSettings().setJavaScriptEnabled(true);
        // 无参数 
        mWebView.loadUrl("javascript:javacalljs()");
        // 有参数
        mWebView.loadUrl("javascript:javacalljswith('http://blog.csdn.net/Leejizhou')");
        // 只需要将第一种方法的loadUrl()换成下面该方法即可
        mWebView.evaluateJavascript("javascript:callJS()", new ValueCallback<String>() {
            @Override
            public void onReceiveValue(String value) {
                //此处为 js 返回的结果
            }
        });
```

# 对于JS调用Android代码的方法有：
1. 通过WebView的addJavascriptInterface()进行对象映射 
2. 通过 WebViewClient 的shouldOverrideUrlLoading()方法回调拦截 url、onPageFinished 在加载完后注入其他代码
3. 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt()方法回调拦截JS对话框alert()、confirm()、prompt() 消息
## 代码展示
```java
        // 给js 提供访问android 的接口
        mWebView.addJavascriptInterface(MainActivity.this, "android");
        // shouldOverrideUrlLoading()方法回调拦截 url 等
        mWebView.setWebViewClient(new MyWebViewClient());
        // 重写OnJsAlert 之类的方法
        mWebView.setWebChromeClient(new MyWebChromeClient());
```
js 调用android 代码示例：
```javascript
<input type="button" value="点击调用java代码" onclick="window.android.startFunction()"/>
```
拦截回调url 注入代码等
```java
    class MyWebViewClient extends WebViewClient {
        boolean loadingFinished;
        boolean redirect;

        @Override
        public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
            String url = request.getUrl().toString();
            Log.d(TAG, "shouldOverrideUrlLoading: rul: " + url);
            if (!loadingFinished) {
                redirect = true;
            }

            loadingFinished = false;
            // mWebView.loadUrl("https:www.bing.com");
            mWebView.loadUrl(request.getUrl().toString());
            return true;
        }

        @Override
        public void onPageStarted(WebView view, String url, Bitmap favicon) {
            Log.d(TAG, "onPageStarted: ");
            super.onPageStarted(view, url, favicon);
            loadingFinished = false;
            //SHOW LOADING IF IT ISNT ALREADY VISIBLE
        }

        @Override
        public void onPageFinished(WebView view, String url) {
            Log.d(TAG, "onPageFinished: ");
            if (!redirect) {
                loadingFinished = true;
            }

            if (loadingFinished && !redirect) {
                //HIDE LOADING IT HAS FINISHED
                imgReset();
// mWebView.loadUrl("javascript:" + "window.alert('Js injection success')");
                Log.d(TAG, "onPageFinished: alert");
            } else {
                redirect = false;
            }
        }
    }
```
方法回调拦截JS对话框
```java
    class MyWebChromeClient extends WebChromeClient {
        @Override
        public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {
            AlertDialog.Builder b = new AlertDialog.Builder(MainActivity.this);
            b.setTitle("Alert");
            b.setMessage(message);
            b.setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm();
                }
            });
            b.setCancelable(false);
            b.create().show();
            return true;
        }
    }
```