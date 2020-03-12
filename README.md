# note4-OkHttp-
基于OkHttp和EventBus封装的网络请求工具类,EventBus主要负责将请求结果发送给相应的界面。
```java
package com.yys.tool;

import java.io.IOException;

import org.greenrobot.eventbus.EventBus;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import android.app.ProgressDialog;
import android.content.Context;
import android.os.Handler;
import android.os.Message;
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.FormBody;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;
/*
 * 基于OkHttp的网络访问工具类
 */
public class OkHttpTools {
	private JSONArray jsonArray;//返回给请求页面的jsonArray
	private JSONObject jsonObject;//返回给数据页面的jsonObject
	private FormBody formBody;//要传给后台的formBody
	private String requestUrl;//请求的链接
	private String requestName;//调用哪个网络请求的名称
	private static Handler handler;
	private String msg="";//用于存放请求成功后返回的消息
	private String code="";//用于存放请求成功后返回的code
	private String [] dataCode;
	private String [] dataName;
	private String result="";//用于存放网络请求成功后，返回的json字符串
	/*
	 * 请求数据
	 */
	public OkHttpTools(String requestName,String requestUrl) {
		this.requestName=requestName;
		this.requestUrl=requestUrl;
		RequestData(requestName);
	}
	/*
	 * 提交数据
	 */
	public OkHttpTools(String requestName,String requestUrl,FormBody formBody) {
		this.requestName=requestName;
		this.requestUrl=requestUrl;
		this.formBody=formBody;
		RequestData(requestName);
		
	}
	private void RequestData(String requestName) {
		new Thread(new RequestThread(requestName)).start();
		handler=new Handler() {
			@Override
			public void handleMessage(Message msg) {
				// TODO Auto-generated method stub
				switch (msg.what) {
				case 0://将获取的数据发送给相应的界面
					EventBus.getDefault().post(new EventBusTools("请求成功",jsonArray));
					break;
				case 1:
					//将数据保存成功的消息发送给相应的界面
					EventBus.getDefault().post(new EventBusTools("数据保存成功"));
					break;
				case 2:
					//将获取的数据发送给相应的界面
					EventBus.getDefault().post(new EventBusTools("请求成功",jsonObject));
					break;
				case 100:
					//网络请求成功，但在解析返回数据的过程中发生错误时的具体代码放这里
					break;
				case 101:
					//网络请求失败时的具体代码放这里
					break;
				default:
					break;
				}
			}
			
		};
	}
	private class RequestThread implements Runnable{
		private String requestType;
		private RequestThread(String reqestType1) {
			requestType=reqestType1;
		}
		@Override
		public void run() {
			// TODO Auto-generated method stub
			String requestStr=requestType;
			if("example1".equals(requestStr)) {//模拟向后台获取数据
				try {
					JSONObject ob=new JSONObject(getResultByGet(requestUrl));
					code=ob.getString("code");
					msg=ob.getString("msg");
					if("0".equals(code)) {
						//后台请求成功的相应操作，
						jsonArray=ob.getJSONArray("data");
						handler.sendEmptyMessage(0);
					}else {
						//后台请求失败的相应操作，
						handler.sendEmptyMessage(100);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
					handler.sendEmptyMessage(100);
				}
			}else if("example2".equals(requestStr)) {//模拟向后台提交数据
				try {
					JSONObject ob=new JSONObject(getResultByPost(requestUrl, formBody));
					code=ob.getString("code");
					msg=ob.getString("msg");
					if("0".equals(code)) {
						//后台请求成功的相应操作，
						handler.sendEmptyMessage(1);
					}else {
						//后台请求失败的相应操作，
						handler.sendEmptyMessage(100);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
					handler.sendEmptyMessage(100);
				}
			}else if("example3".equals(requestStr)) {//模拟向后台提交数据
				try {
					JSONObject ob=new JSONObject(getResultByGet(requestUrl));
					code=ob.getString("code");
					msg=ob.getString("msg");
					if("0".equals(code)) {
						//后台请求成功的相应操作，
						jsonObject=ob.getJSONObject("data");
						handler.sendEmptyMessage(2);
					}else {
						//后台请求失败的相应操作，
						handler.sendEmptyMessage(100);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
					handler.sendEmptyMessage(100);
				}
			}
		}
		
	}
	private String getResultByGet(String url) {
		OkHttpClient client=new OkHttpClient();
		Request request=new Request.Builder().url(url).build();
		Call call=client.newCall(request);
		call.enqueue(new Callback() {
			
			@Override
			public void onResponse(Call call, Response resp) throws IOException {//访问成功
				// TODO Auto-generated method stub
				result=resp.body().string();
			}
			
			@Override
			public void onFailure(Call arg0, IOException arg1) {//网络错误
				// TODO Auto-generated method stub
				handler.sendEmptyMessage(101);
			}
		});
		return result;
	}
	private String getResultByPost(String url,FormBody formBody) {
		OkHttpClient client=new OkHttpClient();
		Request request=new Request.Builder().url(url).post(formBody).build();
		Call call=client.newCall(request);
		call.enqueue(new Callback() {
			
			@Override
			public void onResponse(Call call, Response resp) throws IOException {//访问成功
				// TODO Auto-generated method stub
				result=resp.body().string();
			}
			
			@Override
			public void onFailure(Call arg0, IOException arg1) {//网络错误
				// TODO Auto-generated method stub
				handler.sendEmptyMessage(101);
			}
		});
		return result;
	}
	/*
	 * 网络请求时弹出的进度对话框
	 */
	public ProgressDialog showProgressDialog(Context context,String msg) {
		ProgressDialog dialog=new ProgressDialog(context);
		dialog.setTitle("数据请求中：");
		dialog.setMessage(msg);
		dialog.setCancelable(false);
		dialog.show();
		return dialog;
	}
}
```

