---
Title: Embed WebView in Fragment
Date: 2014-08-29 02:19:26
Tags: webview
---

this comes frome: <http://android-er.blogspot.com/2013/04/embed-webview-in-fragment.html?utm_source=tuicool>

This exercise embed WebView in Fragment.

![Embed WebView in Fragment](http://3.bp.blogspot.com/-E1hJ-lqEJgw/UXwfeztD4mI/AAAAAAAAH1c/nCU6AXVwRbc/s1600/AndroidWebFragment.png)



    package com.example.androidwebviewfragment;

    import android.os.Bundle;
    import android.app.Activity;
    import android.app.Fragment;
    import android.view.LayoutInflater;
    import android.view.View;
    import android.view.ViewGroup;
    import android.webkit.WebView;
    import android.webkit.WebViewClient;
    
    public class MainActivity extends Activity {
    
     static public class MyWebViewFragment extends Fragment {
      
      WebView myWebView;
      final static String myBlogAddr = "http://android-er.blogspot.com";
      String myUrl;
      
    
      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
       View view = inflater.inflate(R.layout.layout_webfragment, container, false);
       myWebView = (WebView)view.findViewById(R.id.mywebview);
       
       myWebView.getSettings().setJavaScriptEnabled(true);                
       myWebView.setWebViewClient(new MyWebViewClient());
       
       if(myUrl == null){
        myUrl = myBlogAddr;
       }
       myWebView.loadUrl(myUrl);
       return view;
      }
      
      private class MyWebViewClient extends WebViewClient {
             @Override
             public boolean shouldOverrideUrlLoading(WebView view, String url) {
              myUrl = url;
                 view.loadUrl(url);
                 return true;
             }
         }
    
      @Override
      public void onActivityCreated(Bundle savedInstanceState) {
       super.onActivityCreated(savedInstanceState);
       setRetainInstance(true);
      }
     }
    
     @Override
     protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
     }
    
     @Override
     public void onBackPressed() {
      MyWebViewFragment fragment = 
        (MyWebViewFragment)getFragmentManager().findFragmentById(R.id.myweb_fragment);
      WebView webView = fragment.myWebView;
      
      if(webView.canGoBack()){
       webView.goBack();
      }else{
       super.onBackPressed();
      }
     }
    }
    

Layout, activity_main.xml.

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin"
        tools:context=".MainActivity" >
    
        <fragment
            android:name="com.example.androidwebviewfragment.MainActivity$MyWebViewFragment"
            android:id="@+id/myweb_fragment"
            android:layout_height="match_parent"
            android:layout_width="match_parent" />

    </RelativeLayout>


Layout of our fragment, layout_webfragment.xml.

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >
    
        <WebView
            android:id="@+id/mywebview"
            android:layout_height="match_parent"
            android:layout_width="match_parent" />

    </LinearLayout>


Permission of "android.permission.INTERNET" is need.

- I can't implement using WebViewFragment, so this exercise extends Fragment, not WebViewFragment.
- In this implement, the WebView can load the last loaded address after orientation changed, but cannot keep the navigation history.