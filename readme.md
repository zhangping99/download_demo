# 文件下载解决中文文件名问题
* 解决思路：
    1. 获取客户端使用的浏览器版本信息
    2. 根据不同的版本信息，设置filename的编码方式不同
* download.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="DownLoadServlet?filename=九尾.jpg">图片1</a>

<a href="DownLoadServlet?filename=1.avi">视频</a>

</body>
</html>
```
* DownLoadUtils
```java
public class DownLoadUtils {
    public static String  getFileName(String agent,String filename) throws UnsupportedEncodingException {
        if(agent.contains("MSIE")){
            //IE
            filename = URLEncoder.encode(filename,"utf-8");
            filename = filename.replace("+"," ");
        }else if (agent.contains("Firefox")){
            //火狐
            BASE64Encoder base64Encoder = new BASE64Encoder();
            filename = "=utf-8?B?"+base64Encoder.encode(filename.getBytes("utf-8"))+"?=";

        }else {
            //其他浏览器
            filename = URLEncoder.encode(filename,"utf-8");
        }

        return filename;

    }
}
```
* DownLoadServlet
```java
package com.zp.servlet;

import com.zp.utils.DownLoadUtils;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.FileInputStream;
import java.io.IOException;

@WebServlet("/DownLoadServlet")
public class DownLoadServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.获取请求参数，文件名称
        String filename = request.getParameter("filename");
        //2.使用字节输入流加载文件进内存
        //2.1找到文件服务器路径
        ServletContext servletContext = getServletContext();
        String realPath = servletContext.getRealPath("/img/" + filename);
        //2.2用字节流关联
        FileInputStream fileInputStream = new FileInputStream(realPath);
        //3.设置response响应头
        //3.1设置响应头类型：content-type
        String mimeType = servletContext.getMimeType(filename);//获取文件的mime类型
        response.setHeader("content-type",mimeType);
        //3.2设置响应头打开方式：content-disposition
        //解决中文文件名问题
        //1.获取user-agent请求头
        String agent = request.getHeader("user-agent");
        //2.使用工具类方法编码文件名即可
        filename = DownLoadUtils.getFileName(agent, filename);
        response.setHeader("content-disposition","attachment;filename="+filename);
        //4.将输入流的数据写出到输出流中
        ServletOutputStream outputStream = response.getOutputStream();
        byte[] buff = new byte[1024*8];
        int len = 0;
        while ((len = fileInputStream.read(buff))!=-1){
            outputStream.write(buff,0,len);
        }
        fileInputStream.close();


    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```