---
Title: Fragment+WebView
Date: 2014-08-29 02:08:10
Tags: webview
---

OK!I can't input any chinese words now!WTF!
This is my first study notes on Android.
Today,oh no,it was yestaday.I have beening search a way to indicat some webpage on a phone screen,and finally, after hours' searching,I found something.


java code:

    public class BaseFragment extends Fragment{
         private WebView webview;
    	@Override
    	public View onCreateView(LayoutInflater inflater, ViewGroup container,
    			Bundle savedInstanceState) {
    		return  inflater.inflate(R.layout.base_fragment, container, false); 
    	}
    
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
    	// TODO Auto-generated method stub
    	 setView(); 
    	 setListener();
    	super.onActivityCreated(savedInstanceState);
    }
    
    private void setListener() {
    	// TODO Auto-generated method stub
    	webview.loadUrl("http://www.zhihudaily.net/section");
    	webview.setWebViewClient(new WebViewClient(){    
    		//overwrite shouldOverrideUrlLoading()
            public boolean shouldOverrideUrlLoading(WebView   view, String url) {       
                view.loadUrl(url);    
                return true;       
            }       
    }); 
    }
    
    private void setView() {
    	// TODO Auto-generated method stub
    	 webview=(WebView)getView().findViewById(R.id.webView1);
    	 }
    }
 
    
base_fragment.xml

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" 
     >
    <WebView
        android:id="@+id/webView1"
        android:layout_width="match_parent"
        android:layout_height="428dp" />

    </LinearLayout>