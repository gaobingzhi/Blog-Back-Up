---
title: Hybrid App 开发模式下的一次揪心历程
date: 2018-04-27 16:47:26
tags: [Bug]
reward: true
---

#### WebView 的痛
![红包](http://pics.blackbirdsport.com/album/201807/1502808_1532681666914.png@180w_320h)

Web App 与 Native App 混合开发模式有着Native App良好的用户体验和Web App 的跨品台优势，所以很是受宠。当然，我们公司也采用了这种模式，采用JsBridge作为两者之间的通信桥梁。WebView可以说是Android最难用的控件了，需要适配Android的各个版本，比如说以下几点：

<!--more-->

###### 一、通过WebView上传文件，就需要继承WebChromeClient

 ```
 protected class CustomWebChromeClient extends WebChromeClient {
        
        //扩展浏览器上传文件
        // For Andorid 3.0++
        public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType) {
            openFileChooserImpl(uploadMsg);
        }
        
        //For Andorid 3.0--
        public void openFileChooser(ValueCallback<Uri> uploadMsg) {
            openFileChooserImpl(uploadMsg);
        }
        
        public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
            openFileChooserImpl(uploadMsg);
        }
        
        // For Android > 5.0
        @Override
        public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
            openFileChooserImplForAndroid5(filePathCallback);
            return true;
        }
        
        @Override
        public void onProgressChanged(WebView view, int newProgress) {
            super.onProgressChanged(view, newProgress);
        }
        
        @Override
        public void onReceivedTitle(WebView view, String mTitle) {
            super.onReceivedTitle(view, mTitle);
            if (StringUtil.isEmpty(title)) {
                wvTitleView.setText(mTitle);
            }
        }
    }
 ```
 
 同时还需要注意在onActivityResult回调中调用ValueCallback<Uri> 或ValueCallback<Uri[]> 的onReceiveValue(null) 方法。
 
 ###### 二、CustomWebViewClient 设置

```
private class CustomWebViewClient extends WVJBWebViewClient {
        String errorPath = "file:///";
        
        private CustomWebViewClient(WVJBWebView webView) {
            super(webView);
        }
        
        @Override
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            LogUtil.e("url", url);
            return super.shouldOverrideUrlLoading(view, url);
        }
        
        @Override
        public void onPageStarted(WebView view, String url, Bitmap favicon) {
            super.onPageStarted(view, url, favicon);
            LogUtil.e("onPageStarted---", url);
            mProgressBar.setVisibility(View.VISIBLE);
            mProgressBar.setAlpha(1.0f);
            subPageStarted();
        }
        
        @Override
        public void onPageFinished(WebView view, String url) {
            super.onPageFinished(view, url);
            if (rightButton.getVisibility() == View.GONE) {
                setRightButtonVisibility(View.VISIBLE);
            }
            subPageFinished();
        }
        
        @Override
        public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
            if (errorLayout.getVisibility() != View.VISIBLE) {
                wvjbWebView.setVisibility(View.GONE);
                errorLayout.setVisibility(View.VISIBLE);
            }
            
            subReceivedError();
        }
        
        @Override
        public void onLoadResource(WebView view, String url) {
            super.onLoadResource(view, url);
        }
    }
```
 
###### 三、关于WebSettings

```
protected void initSet() {
        wvjbWebView.setWebChromeClient(new CustomWebChromeClient());
        wvjbWebView.setWebViewClient(new CustomWebViewClient(wvjbWebView));
    	//是否打开debug模式
        //WebView.setWebContentsDebuggingEnabled(true);
        WebSettings webSettings = wvjbWebView.getSettings();
        
        //如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript
        webSettings.setJavaScriptEnabled(true);
        
        //设置自适应屏幕，两者合用
        webSettings.setUseWideViewPort(true);
        //将图片调整到适合webview的大小
        webSettings.setLoadWithOverviewMode(true);
        // 缩放至屏幕的大小
        webSettings.setRenderPriority(WebSettings.RenderPriority.HIGH);
        
        //设置缓存
        webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);
        webSettings.setDomStorageEnabled(true);
        webSettings.setAppCacheEnabled(true);
        if (!ShareUtils.getWebFirstSet(this)) {
            ShareUtils.setWebFirstSet(this, true);
            String cacheDirPath = getApplicationContext().getDir("cache", Context.MODE_PRIVATE).getPath();
            webSettings.setAppCachePath(cacheDirPath);
            webSettings.setAppCacheMaxSize(8 * 1024 * 1024);
        }
        
        webSettings.setBlockNetworkLoads(false);
        webSettings.setBlockNetworkImage(false);
        
        webSettings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.SINGLE_COLUMN);
        webSettings.supportMultipleWindows();
        webSettings.setAllowFileAccess(true);
        webSettings.setNeedInitialFocus(true);
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);
        webSettings.setDefaultTextEncodingName("UTF-8");
        if (Build.VERSION.SDK_INT >= 19) {
            webSettings.setLoadsImagesAutomatically(true);
        } else {
            webSettings.setLoadsImagesAutomatically(false);
        }
        
        //设置Http与Https的混合模式
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
        }
        
        String ua = webSettings.getUserAgentString();
        webSettings.setUserAgentString(ua + "....");
    }
```

#### JsBridge 初始化失败，红包打不开的问题

###### 一、确认Bug产生机型
Android or iPhone ，如果是Android，确定Android品牌，Bug产生初期，反应的机型都为Vivo手机，奈何身边没有Vivo机型。以为就是Vivo某些机型下产生的Bug。（测试样例不够，不能确定产生的机型）
###### 二、确定测试环境
WiFi or 移动网络，移动网络下存在WebView被植入Js的问题，可能会破坏JsBridge桥梁，导致Web与Native 不能进行通行，从而打不开红包。通过测试样例发现，无论WiFi下还是移动网络下都存在红包打不开的问题。确定不是因为流氓广告商通过运营商植入Js脚本导致JsBridge通信不了的问题。
###### 三、Bug产生的机型不止Vivo机型
随着Bug产生的数量较多，反应用户的机型也不尽相同，魅族、OPPO、华为等机型下也有相同问题。
###### 四、寻找Bug机型
根据测试的情况反应，应该是Web 内部产生错误或异常，导致JsBridge模块失败，从而产生通信不了。

寻找产生问题的机型（Bug复现），但很难找到问题机型。导致问题不能及时解决。
###### 五、发现错误，解决问题
终于，苦心人太不负，找到一款Android 5.0的华为手机，存在问题。通过Chrome浏览器查看执行过程（chrome://inspect/#devices），发现问题发生在Js的一个变量的新的定义方式在一些低版本的浏览器不识别，导致Js错误，从而导致JsBridge初始化错误。修改错误，解决问题。

##### 总结
在Debug过程中，需要考虑周全，确定问题产生范围，在此条件下保持单次验证变量唯一性，不断的缩小问题范围，在没有可更改的变量情况下，需要大胆的猜想与积极的心态，寻找突破口，复现问题产生的场景，进而才能解决问题。