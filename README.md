# Api-Gateway-Demo-Android 使用指南
## 1 SDK简介

本SDK是阿里云CloudApi团队提供给用户的调用示例代码，代码文件的层级结构如下：

**SDK实现文件**  

* com.alibaba.cloudapi.client
	* HttpUtil		`Http工具类`
	* SignUtil		`签名的实现类`
	* AppConfiguration	`所有SDK的配置设置`
	* com.alibaba.cloudapi.client.constant
		* ContentType	`常用HTTP Content-Type常量`
		* HttpHeader	`HTTP头常量`
		* HttpMethod	`HTTP方法常量`
		* HttpSchema	`HTTP Schema常量`
		* Constants		`通用常量`
		* SystemHeader	`系统HTTP头常量`

**SDK调用文件**  

* com.alibaba.cloudapi.client	`调用示例`
	* MainActivity		`HTTP请求发送页面`
	* DisplayMessageActivity		`显示服务器应答页面`

本SDK实现了CloudApi的签名算法和一系列Http调用示例，可以在MainActivity.java看到具体调用办法：  

* Http Get  
* Http Post Form
* Http Post Bytes
* Http Put Bytes
* Http Delete

其中SDK的签名算法比较复杂，如果想了解其中的算法细节，请到此网址查看：
`https://help.aliyun.com/document_detail/29475.html

## 2 SDK配置

本SDK需要配置好之后才能正常使用，在AppConfiguration中，有三个值是必须配置的：

    /**
     *  Api绑定的的AppKey，可以在“阿里云官网”->"API网关"->"应用管理"->"应用详情"查看
     */
    public static final String APP_KEY = "";

    /**
     *  Api绑定的的AppSecret，用来做传输数据签名使用，可以在“阿里云官网”->"API网关"->"应用管理"->"应用详情"查看
     */
    public static final String APP_SECRET = "";


    /**
     *  Api绑定的域名
     */
    public static final String APP_HOST = "";

**重要提示：APP_KEY和APP_SECRET是网关认证用户请求的钥匙，这两个配置如果保存在客户端，需要加密处理。** 

另外有一个配置需要注意一下，如果Api提供商只提供HTTPS方式的调用，那么需要修改配置中的IS_HTTPS为true。Sdk将以忽略证书的方式来发起HTTPS请求，如果你能拿到服务器通商提供的证书，建议你修改HttpUtil类来采取SSL证书方式发起HTTPS请求。

    /**
     * 是否以HTTPS方式提交请求
     * 本SDK采取忽略证书的模式,目的是方便大家的调试
     * 为了安全起见,建议采取证书校验方式
     */
    public static final boolean IS_HTTPS = true;

## 3 SDK使用

### 3.1 把SDK引入你的项目
配置完成之后，本SDK就可以使用了，建议做代码级应用，这样便于使用者调试理解SDK的工作原理。直接把com.alibaba.cloudapi.client文件夹复制到你项目的src/java文件夹下，然后就能直接应用了。


### 3.2 工程中引入头文件

	import com.alibaba.cloudapi.client.*
	import com.alibaba.cloudapi.client.constant*
	
### 3.3 SDK依赖的开源库

本SDK依赖了okhttp3开源组件，版本号是3.4.1，请引用sdk代码的时候，务必注意在你项目的build.gradle中dependencies中配置okhttp组件：


	apply plugin: 'com.android.application'
	
	android {
	    compileSdkVersion 24
	    buildToolsVersion "24.0.2"
	
	    defaultConfig {
	        applicationId "com.alibaba.cloudapi.client"
	        minSdkVersion 15
	        targetSdkVersion 24
	        versionCode 1
	        versionName "1.0"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	}
	
	dependencies {
	    compile fileTree(include: ['*.jar'], dir: 'libs')
	    testCompile 'junit:junit:4.12'
	    compile 'com.android.support:appcompat-v7:24.2.0'
	    compile 'com.squareup.okhttp3:okhttp:3.4.1'
	}
	

### 3.4 调用Demo

然后可以像MainActivity.java中的各个方法一样调用HttpUtil类发送各种类型的Http请求。  
**需要特别注意的是，如果网关检测到请求不合法，或者网关、后端服务器出现故障，错误描述信息会在HTTP应答的名字为X-Ca-Error-Message头信息中返回，部分常见的错误信息可以在下面这个网址看到详细解释：  
https://help.aliyun.com/document_detail/43800.html**

#### 3.4.1 Http Get

	public void sendHttpGet(View view) {

        final Intent intent = new Intent(this, DisplayMessageActivity.class);

        Runnable runnable = new Runnable(){
            @Override
            public void run() {
                String getPath = "/v3/getUserTest/[userId]";
                Map<String , String> getPathParams = new HashMap<>();
                getPathParams.put("userId" , "10000003");

                Map<String , String> getQueryParams = new HashMap<>();
                getQueryParams.put("sex" , "男");

                Map<String , String> getHeaderParams = new HashMap<>();
                getHeaderParams.put("age" , "18");

                HttpUtil.getInstance().httpGet(getPath , getPathParams , getQueryParams , getHeaderParams , new Callback() {
                    @Override
                    public void onFailure(Call call, IOException e) {
                        intent.putExtra(EXTRA_MESSAGE , e.getMessage());
                        startActivity(intent);
                    }

                    @Override
                    public void onResponse(Call call, Response response) throws IOException {
                    	StringBuilder result = new StringBuilder();
        				result.append("【服务器返回结果为】").append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				result.append("ResultCode:").append(Constants.CLOUDAPI_LF).append(response.code()).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				if(response.code() != 200){
            				result.append("错误原因：").append(response.header("X-Ca-Error-Message")).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				}

        				result.append("ResultBody:").append(Constants.CLOUDAPI_LF).append(new String(response.body().bytes() , Constants.CLOUDAPI_ENCODING));

                        intent.putExtra(EXTRA_MESSAGE , result.toString());
                        startActivity(intent);
                    }
                });
            }
        };
        new Thread(runnable).start();
    }
	
#### 3.4.2 Http Post Form

	public void sendHttpPostForm(View view) {

        final Intent intent = new Intent(this, DisplayMessageActivity.class);

        Runnable runnable = new Runnable(){
            @Override
            public void run() {
                String getPath = "/testhttp/postform";

                Map<String , String> postQueryParams = new HashMap<>();
                postQueryParams.put("sex" , "男");

                Map<String , String> postHeaderParams = new HashMap<>();
                postHeaderParams.put("TestPostHeader" , "HTTP头");

                Map<String , String> postFormParams = new HashMap<>();
                postFormParams.put("TestAccount" , "FORM表单");


                HttpUtil.getInstance().httpPostForm(getPath , null , postQueryParams , postFormParams,  postHeaderParams , new Callback() {
                    @Override
                    public void onFailure(Call call, IOException e) {
                        intent.putExtra(EXTRA_MESSAGE , e.getMessage());
                        startActivity(intent);
                    }

                    @Override
                    public void onResponse(Call call, Response response) throws IOException {
                        StringBuilder result = new StringBuilder();
        				result.append("【服务器返回结果为】").append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				result.append("ResultCode:").append(Constants.CLOUDAPI_LF).append(response.code()).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				if(response.code() != 200){
            				result.append("错误原因：").append(response.header("X-Ca-Error-Message")).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				}

        				result.append("ResultBody:").append(Constants.CLOUDAPI_LF).append(new String(response.body().bytes() , Constants.CLOUDAPI_ENCODING));

                        intent.putExtra(EXTRA_MESSAGE , result.toString());

                        startActivity(intent);


                    }
                });

            }
        };
        new Thread(runnable).start();
    }


#### 3.4.3 Http Post Bytes

	public void sendHttpPostBytes(View view) {

        final Intent intent = new Intent(this, DisplayMessageActivity.class);

        Runnable runnable = new Runnable(){
            @Override
            public void run() {
                String getPath = "/testhttp/postbyte";

                Map<String , String> getQueryParams = new HashMap<>();
                getQueryParams.put("QueryName" , "TestQueryNameVlaue");

                Map<String , String> getHeaderParams = new HashMap<>();
                getHeaderParams.put("TestPostHeader" , "HTTP头");

                String content =  "Hi there ,this is 一个 string";

                HttpUtil.getInstance().httpPostBytes(getPath , null , getQueryParams , content.getBytes(Constants.CLOUDAPI_ENCODING) , getHeaderParams , new Callback() {
                    @Override
                    public void onFailure(Call call, IOException e) {
                        intent.putExtra(EXTRA_MESSAGE , e.getMessage());
                        startActivity(intent);
                    }

                    @Override
                    public void onResponse(Call call, Response response) throws IOException {
                        StringBuilder result = new StringBuilder();
        				result.append("【服务器返回结果为】").append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				result.append("ResultCode:").append(Constants.CLOUDAPI_LF).append(response.code()).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				if(response.code() != 200){
            				result.append("错误原因：").append(response.header("X-Ca-Error-Message")).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				}

        				result.append("ResultBody:").append(Constants.CLOUDAPI_LF).append(new String(response.body().bytes() , Constants.CLOUDAPI_ENCODING));

                        StringBuilder result = new StringBuilder();
        				result.append("【服务器返回结果为】").append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				result.append("ResultCode:").append(Constants.CLOUDAPI_LF).append(response.code()).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				if(response.code() != 200){
            				result.append("错误原因：").append(response.header("X-Ca-Error-Message")).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				}

        				result.append("ResultBody:").append(Constants.CLOUDAPI_LF).append(new String(response.body().bytes() , Constants.CLOUDAPI_ENCODING));

                        intent.putExtra(EXTRA_MESSAGE , result.toString());
                        startActivity(intent);
                    }
                });

            }
        };
        new Thread(runnable).start();
    }

	
####3.4.4 Http Put Bytes

	public void sendHttpPutBytes(View view) {

        final Intent intent = new Intent(this, DisplayMessageActivity.class);

        Runnable runnable = new Runnable(){
            @Override
            public void run() {
                String getPath = "/testhttp/putform";


                Map<String , String> getQueryParams = new HashMap<>();
                getQueryParams.put("QueryName" , "TestQueryNameVlaue");

                Map<String , String> getHeaderParams = new HashMap<>();
                getHeaderParams.put("TestPostHeader" , "TestPostHeaderValue");

                String content =  "Hi there ,this is 一个 string";

                HttpUtil.getInstance().httpPutBytes(getPath , null , getQueryParams  , content.getBytes(Constants.CLOUDAPI_ENCODING) , getHeaderParams , new Callback() {
                    @Override
                    public void onFailure(Call call, IOException e) {
                        intent.putExtra(EXTRA_MESSAGE , e.getMessage());
                        startActivity(intent);
                    }

                    @Override
                    public void onResponse(Call call, Response response) throws IOException {
                        StringBuilder result = new StringBuilder();
        				result.append("【服务器返回结果为】").append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				result.append("ResultCode:").append(Constants.CLOUDAPI_LF).append(response.code()).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				if(response.code() != 200){
            				result.append("错误原因：").append(response.header("X-Ca-Error-Message")).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				}

        				result.append("ResultBody:").append(Constants.CLOUDAPI_LF).append(new String(response.body().bytes() , Constants.CLOUDAPI_ENCODING));

                        intent.putExtra(EXTRA_MESSAGE , result.toString());
                        startActivity(intent);

                    }
                });

            }
        };
        new Thread(runnable).start();
    }
	
#### 3.4.5 Http Put Bytes
	
	public void sendHttpDelete(View view) {

        final Intent intent = new Intent(this, DisplayMessageActivity.class);

        Runnable runnable = new Runnable(){
            @Override
            public void run() {
                String deletePath = "/testhttp/delete";

                Map<String , String> deleteQueryParams = new HashMap<>();
                deleteQueryParams.put("QueryName" , "TestQueryNameVlaue");

                Map<String , String> deleteHeaderParams = new HashMap<>();
                deleteHeaderParams.put("TestDelete" , "TestDeleteValue");

                HttpUtil.getInstance().httpDelete(deletePath , null , deleteQueryParams , deleteHeaderParams , new Callback() {
                    @Override
                    public void onFailure(Call call, IOException e) {
                        intent.putExtra(EXTRA_MESSAGE , e.getMessage());
                        startActivity(intent);
                    }

                    @Override
                    public void onResponse(Call call, Response response) throws IOException {
                        StringBuilder result = new StringBuilder();
        				result.append("【服务器返回结果为】").append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				result.append("ResultCode:").append(Constants.CLOUDAPI_LF).append(response.code()).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				if(response.code() != 200){
            				result.append("错误原因：").append(response.header("X-Ca-Error-Message")).append(Constants.CLOUDAPI_LF).append(Constants.CLOUDAPI_LF);
        				}

        				result.append("ResultBody:").append(Constants.CLOUDAPI_LF).append(new String(response.body().bytes() , Constants.CLOUDAPI_ENCODING));

                        intent.putExtra(EXTRA_MESSAGE , result.toString());

                        startActivity(intent);


                    }
                });

            }
        };
        new Thread(runnable).start();
    }
	
如果在使用中遇到棘手的问题，请加入我们官方旺旺群来找我们，群号：1640106170