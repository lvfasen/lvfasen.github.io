HttpURLConnection

```java
package com.url;

import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLConnection;

public class UrlTest {

  public static void main(String[] args) {
    URLConnection uc = null;
    InputStream input = null;
    BufferedInputStream bufferInput = null;
    try {
      URL url = new URL("http://www.baidu.com");
      uc = url.openConnection();
      HttpURLConnection urlConnection = (HttpURLConnection)uc;
      urlConnection.setRequestMethod("get");
      input = uc.getInputStream();
      bufferInput = new BufferedInputStream(input);

      InputStreamReader reader = new InputStreamReader(bufferInput);

      int i;
      if((i = reader.read()) > 0 ){
        System.out.println((char) i);
      }
    } catch (IOException e) {
      e.printStackTrace();
    } finally {
      try {
        input.close();
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }


  public static String sendPostRequest(String url,String param){

    HttpURLConnection httpURLConnection = null;

    OutputStream out = null; //写

    InputStream in = null; //读

    int responseCode = 0; //远程主机响应的HTTP状态码

    String result="";

    try{

      URL sendUrl = new URL(url);

      httpURLConnection = (HttpURLConnection)sendUrl.openConnection();

      //post方式请求

      httpURLConnection.setRequestMethod("POST");

      //设置头部信息

      httpURLConnection.setRequestProperty("headerdata", "ceshiyongde");

      //一定要设置 Content-Type 要不然服务端接收不到参数

      httpURLConnection.setRequestProperty("Content-Type", "application/Json; charset=UTF-8");

      //指示应用程序要将数据写入URL连接,其值默认为false(是否传参)

      httpURLConnection.setDoOutput(true);

      //httpURLConnection.setDoInput(true);

      httpURLConnection.setUseCaches(false);

      httpURLConnection.setConnectTimeout(30000); //30秒连接超时

      httpURLConnection.setReadTimeout(30000); //30秒读取超时

      //获取输出流

      out = httpURLConnection.getOutputStream();

      //输出流里写入POST参数

      out.write(param.getBytes());

      out.flush();

      out.close();

      responseCode = httpURLConnection.getResponseCode();

      BufferedReader br = new BufferedReader(new InputStreamReader(httpURLConnection.getInputStream(),"UTF-8"));

      result =br.readLine();

    }catch(Exception e) {

      e.printStackTrace();

    }

    return result;

  }

}

```

