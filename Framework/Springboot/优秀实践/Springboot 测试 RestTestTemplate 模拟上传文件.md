#JUnit #Springboot #Uploadfile 

# 本地文件上传

```java
 /**
     * 本地文件上传
     * @throws FileNotFoundException
     */
    @Test
    public void localPost() throws FileNotFoundException {
    	//fastdfs地址，文件上传地址
	    private String url = "http://xxx.xxx.xxx.xxx:8188/uploadfile4picture";
	    //文件路径
	    private String filePath = "C:\\Users\\rwto\\Desktop\\测试\\图片\\a.jpg";
        RestTemplate rest = new RestTemplate();
        //获取本地文件
        FileSystemResource resource = new FileSystemResource(new File(filePath));
        //将本地文件编辑为formdata格式的数据
        MultiValueMap<String,Object> param = new LinkedMultiValueMap<String,Object>();
        param.add("file", resource);

        //发送post请求，并获取返回信息
        String string = rest.postForObject(url, param, String.class);
        //String转json格式，获取返回的url地址
        JSONObject jsonObject = JSONObject.parseObject(string);
        JSONObject data = JSONObject.parseObject(jsonObject.getString("data"));
        System.out.println(data.getString("file_uri"));
    }
```


# 通过文件流上传
```java

   /**
     * 文件流上传
     * @throws IOException
     */
    @Test
    public  void streamPost() throws IOException {
		//fastdfs地址，文件上传地址
	    private String url = "http://xxx.xxx.xxx.xxx:8188/uploadfile4picture";
	    //文件路径
	    private String filePath = "C:\\Users\\rwto\\Desktop\\测试\\图片\\a.jpg";
        RestTemplate rest = new RestTemplate();
        //获取文件流
        final FileInputStream in = new FileInputStream(new File(filePath));
        //新建流资源，必须重写contentLength()和getFilename()
        Resource resource = new InputStreamResource(in){
            //文件长度,单位字节
            @Override
            public long contentLength() throws IOException {
                long size = in.available();
                return size;
            }
            //文件名
            @Override
            public String getFilename(){
                return "a.jpg";
            }
        };
        //将本地文件编辑为formdata格式的数据
        MultiValueMap<String,Object> param = new LinkedMultiValueMap<String,Object>();
        param.add("file", resource);

        //发送post请求，并获取返回信息
        String string = rest.postForObject(url, param, String.class);
        //String转json格式，获取返回的url地址
        JSONObject jsonObject = JSONObject.parseObject(string);
        JSONObject data = JSONObject.parseObject(jsonObject.getString("data"));
        System.out.println(data.getString("file_uri"));
    }
```

# 参考
1. [Java借助RestTemplate 模拟发送formdata请求](https://blog.csdn.net/ren9436/article/details/120039521)