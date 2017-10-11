只看看restful的接口吧，其他的再说。
    网络被设计为一个可以访问的文件系统，每一个网络上的资源都有唯一的访问路径。
get请求用来获取资源，  post发送需要服务器处理的数据。
    往    String url = "http://localhost:8161/api/message/STOCKS/test-topic-1?type=topic";
        
这个地址发送post请求：
String result = null;
        DefaultHttpClient client = new DefaultHttpClient();
        client.getCredentialsProvider().setCredentials(new AuthScope("localhost", 8161),
                new UsernamePasswordCredentials("admin", "admin"));
//        client.getParams().setParameter(ConnRouteParams.DEFAULT_PROXY, proxy);
        HttpPost httpPost = new HttpPost(uri);
        List<NameValuePair> nvps = new ArrayList<NameValuePair>();
        
        try {
//            httpPost.setEntity(new UrlEncodedFormEntity(nvps, HTTP.UTF_8));
            httpPost.setEntity(new StringEntity(data))
这个;  