---
title: Retrofit使用总结
date: 2016-08-19 22:30:51
categories: Android
tags: retrofit
---
这篇博客介绍和记录了一些Retrofit的操作
<!-- more -->
## Retrofit简介
Retrofit非常适合于Restful url格式的请求，使用注解的方式提供功能。
Retrofit官方地址：[https://github.com/square/retrofit/](https://github.com/square/retrofit/)
Retrofit文档地址：[http://square.github.io/retrofit/](http://square.github.io/retrofit/)
转换器依赖： [converter-gson](https://mvnrepository.com/artifact/com.squareup.retrofit2/converter-gson)
RxJava适配器依赖：[adapter-rxjava](https://mvnrepository.com/artifact/com.squareup.retrofit2/adapter-rxjava)
OkHttp日志拦截器依赖：[OkHttp Logging Interceptor](https://mvnrepository.com/artifact/com.squareup.okhttp3/logging-interceptor)
RxJava地址：[RxGithub](https://github.com/ReactiveX/RxJava)
RxJava依赖地址：[RxJava依赖](http://search.maven.org/#search%7Cga%7C1%7Cio.reactivex.rxjava)
RxAndroid地址：[RxAndroidGithub](https://github.com/ReactiveX/RxAndroid)
RxAndroid依赖地址：[RxAndroid依赖](http://search.maven.org/#search%7Cga%7C1%7Crxandroid)
## Retrofit使用
### 添加依赖
由于需要进行联网，所以需要在 AndroidManifest.xml文件中 添加网络权限：
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
在官方地址中查看最新的版本，最新版本依赖于 OKHttp,而且这部分不再支持替换。所以不需要显式的导入OkHttp,在 Retrofit 中已经导入 OkKhttp3
添加retrofit依赖
```gradle
compile 'com.squareup.retrofit2:retrofit:2.1.0'
```
添加转换器依赖
```gradle
compile 'com.squareup.retrofit2:converter-gson:2.0.2'//添加转换器
```
### 封装数据
例如返回的数据格式如下
```json
{
  "error": false,
  "results": [
    {
      "_id": "579ab0a8421aa90d36e960b4",
      "createdAt": "2016-07-29T09:26:00.838Z",
      "desc": "7.29",
      "publishedAt": "2016-07-29T09:37:39.219Z",
      "source": "chrome",
      "type": "福利",
      "url": "http://ww3.sinaimg.cn/large/610dc034jw1f6aipo68yvj20qo0qoaee.jpg",
      "used": true,
      "who": "代码家"
    }
  ]
}
```
根据返回的数据格式抽取基类
```java
//泛型代指results字段
public class BaseModel<T> {
    public boolean error;
    public T results;
}
```
然后封装Bean类
```java
public class Benefit {

  public String _id;
  public String createdAt;
  public String desc;
  public String publishedAt;
  public String source;
  public String type;
  public String url;
  public boolean used;
}

```
### 3.使用
#### 1. URL定义
使用Retofit时URL地址由两部分拼接而成，由baseUrl 和接口中 "/"位置的不同具有不同的含义，通常baseUrl要以"/" 结尾，而在Api接口中不以 "/"开头和结尾，才能得到正确的地址。
```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/v3/")
    .build();

interface GitHubService {
  @GET("/repos/{owner}/{repo}/contributors")
  Call<List<Contributor>> repoContributors(
      @Path("owner") String owner,
      @Path("repo") String repo);
}
//解析的URL
//https://api.github.com/repos/square/retrofit/contributors
```
因为在接口中定义的路径以"/"开头，表示绝对地址的后缀路径，所有最终拼接出来的主机地址中没有v3。
如果有前缀"/"就代表着是一个绝对路径。删除了那个前缀的"/"， 将会得到正确的、包含了 v3 路径的全 URL
```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/v3/")
    .build();

interface GitHubService {
  @GET("repos/{owner}/{repo}/contributors")
  Call<List<Contributor>> repoContributors(
      @Path("owner") String owner,
      @Path("repo") String repo);
}

// https://api.github.com/v3/repos/square/retrofit/contributors 
```
#### 定义接口
@Path表示动态Url
```java
public interface Api {
    @GET("api/data/福利/{number}/{page}")
    Call<BaseModel<ArrayList<Benefit>>> defaultBenefits(
            @Path("number") int number,
            @Path("page") int page
    );
}
```
#### 实现请求
```java
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://gank.io/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
Api api = retrofit.create(Api.class);
Call<BaseModel<ArrayList<Benefit>>> call = api.defaultBenefits(20, 1);
//call.execute()同步
//call.enqueue()异步
call.enqueue(new Callback<BaseModel<ArrayList<Benefit>>>() {
        @Override
        public void onResponse(Call<BaseModel<ArrayList<Benefit>>> call, Response<BaseModel<ArrayList<Benefit>>> response) {
            // TODO
            BaseModel<ArrayList<Benefit>>> dataList = response.body().reuslts;
            }

        @Override
        public void onFailure(Call<BaseModel<ArrayList<Benefit>>> call, Throwable t) {
                // TODO
            }
        });
```

无论同步操作还是异步操作，每个Call对象实例之能被执行一次，多次执行会抛出 IllegalStateException:Alredy executed，异常，如果需要多次执行同一个Call对象实例，可以使用Clone()方法克隆一个对象
```java
Api api = retrofit.create(Api.class);
//只能被执行一次
Call<BaseModel<ArrayList<Benefit>>> call = api.defaultBenefits(20, 1);
Call<BaseModel<ArrayList<Benefit>>> cloneCall = call.clone();
//cloneCall.execute() 同步
//cloneCall.enqueue() 异步
```
除了以上动态Url请求外，还有其他注解表示不同的请求

#### 设置查询参数
@Query设置查询参数
```java
//http://baseurl/users?sortby=username
//查询参数只能使用注解@Query，而不能使用注解@Path改写为
//@GET("users?sortby={username}")
//Call<BaseModel<Arraylist<User>>> queryUser(@Path("username") String userName);
public interface Api {
    @GET("users")
    Call<BaseModel<ArrayList<User>>> queryUser(
            @Query("sortby") String username
    );
}
```
#### 设置请求头
**静态添加**
```java
public interface Api{
    @Headers({
        "Accept: application/vnd.github.v3.full+json",
        "User-Agent: RetrofitBean-Sample-App",
        "name:"wptdxii"
    })
    @GET("users/{name})
    Call<BaseModel<User>> getUser(@Path("name") String name);
}
```
**动态添加**
```java
public interface Api {
    @GET("users/{name})
    Call<BaseModel<User>> getUser(
        @Header("Accept") String accept,
        @Header("User-Agent") String agent,
        @Header("name") String name,
        @Path(""name) String name
    );
}
```
可以使用@Header注解来动态更新请求头。并且必须向@Header提供相应的参数。如果参数值为空, 则该请求头将会被忽略。如果参数值不为空, 则会调用该值的toString并使用其结果。使用静态或者动态添加请求头不会相互覆盖，所有同名的请求头都会包含在请求中

**统一请求头**
当需要使用同一请求头时需要为Okhtpp设置拦截器
创建请求头拦截器
```java
public class HeaderInterceptor implements Interceptor {
    @Override
    public Response intercept(Interceptor.Chain chain) throws IOException {
        Request original = chain.request();

        Request request = original.newBuilder()
                //使用header会重写请求头
               // .header("Token", "token1")
               // .header("Token", "token2")
               //使用addHeader不会重写请求头
               // .addHeader("Token", "token1")
               // .addHeader("Token", "token2")
                .addHeader("Accept", "application/vnd.github.v3.full+json")
                .addHeader("User-Agent", "C-RetrofitBean-Sample-App")
                .addHeader("name", "wptdxii") //缩放比 1/2/3
                .build();

        return chain.proceed(request);
    }
}
```
然后将拦截器设置给OkHttp
#### 直接请求URL
当URl不满足RESTFUL类型或者baseUrl不同时，可以传入完整URL
```java
public interface Api{
    @GET
    Call<BaseMode<User>> get(@Url String url);
}
```
#### 实现POST请求
向服务器上传json字符串数据
```java
//这里直接传入对象，内部通过converter自动转换为字符串
public interface Api{
    @POST("add")
    Call<BaseModel<ArrayList<User>>> addUser(@Body User user);
}
```

#### 实现表单上传
@FormUrlEncoded注解表示上传键值对，用来发送表单数据
@Field注解和参数来指定每个表单项的Key，value为参数的值
```java
public interface Api{
    @FormUrlEncoded
    @POST("user/edit")
    Call<BaseModel<User>> update(@Field("first_name") String firstName, @Field("last_name") String lastName);
}
```
当有很多个表单参数时
@FieldMap注解和Map对象参数来指定每个表单项的Key，value的值
```java
public interface Api{
    @FormUrlEncoded
    @POST("user/edit")
    Call<BaseModel<Uer>> update(@FieldMap Map<String,String> fieldMap);
}
```
#### 实现单文件上传
@Multipart表示允许使用多个@Part
@MultipartBody.Part表示要上传的文件
@Part表示键值对
```java
public interface Api {
    @Multipart
    @POST("register")
    Call<User> registUser(@Part MultipartBody.Part photo, @Part("username") RequestBody username, @Part("password") RequestBody password);
}
```
使用方式
```java
File file = new File(Environment.getExternalStorageDirectory,"icon.png");
RequestBody photoRequestBody = RequestBody.create(MediaType.parse("image/png"),file);
//第一个参数表示服务器对应的key，第二个参数表示服务器对应的文件名，第三个参数表示文件
MultipartBody.Part photo =
//可以动态设置文件名
MultipartBody.Part.createFromeData("phtots","icon.png", photoRequestBody);

Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://gank.io/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
Api api = retrofit.create(Api.class);
Call<User> call = api.registUser(photo, RequestBody.create(null, "admin"), RequestBody.create(null, "123"));
//TODO
```
@MultipartBody.Part表示上传的文件，如果要使用@Part RequestBody的方式表示上传的文件，则需要改写为
```java
public interface Api{
    @Multipart
    @Post("api/Accounts/editcount")
    //文件名采用了硬编码，不能动态指定
    Call<BaseModel<User>> login(@Part("file\":filename=\"icon.png") RequestBody file,@Part("name") RequestBody name, @Part("id") RequestBody id);
}
```
这种方式上传的文件名采用了硬编码，不能动态指定
使用方式
```java
File file = new File(Environment.getExternalStorageDirectory,"icon.png");
RequestBody photo = RequestBody.create(MediaType.parse("image/png"),file);

Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://gank.io/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
Api api = retrofit.create(Api.class);
Call<User> call = api.registUser(photo, RequestBody.create(null, "admin"), RequestBody.create(null, "123"));
//TODO
```
#### 实现多文件上传
@PartMap用于标识Map，
```java
public interface Api{
    @Mulipart
    @Post("register")
    Call<BaseModel<User>> regist(@PartMap Map<String, RequestBody> params, @Part("password") ReqeustBody password);
}
```
使用方式
```java
File file = new File(Environment.getExternalStorageDirectory(), "message.png");
        RequestBody photo = RequestBody.create(MediaType.parse("image/png", file);
Map<String,RequestBody> photos = new HashMap<>();
photos.put("photos\"; filename=\"icon.png", photo);
photos.put("username",  RequestBody.create(null, "admin"));

Call<User> call = userBiz.registerUser(photos, RequestBody.create(null, "123"));
//TODO
```
Map可以存放一个或者多个文件，也可以存放简单的键值对，当然在上边示例中，也可以将用户名键值对单独使用@Part，仅将将上传的文件放入Map中，上传文件的键值为("photos\";filename=\"icon.png"),其中photos表示服务器对应的key，filename表示上传到服务器的文件名，支持动态设置，如将本地message.png重命名为icon.png

#### 实现下载
```java
public interface Api{
    @GET("download")
    Call<ResponseBody> download();
}
```
使用方式
```java
call.enqueue(new Callback<ResponseBody>()
{
    @Override
    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response)
    {
        InputStream is = response.body().byteStream();
        //TODO save file
        //切换到子线程进行io操作
    }

    @Override
    public void onFailure(Call<ResponseBody> call, Throwable t)
    {
        //TODO
    }
});
```
onReponse回调虽然在UI线程，但是还是要处理io操作，在这里还要另外开线程操作，所有直接并不适合进行下载，可以使用OKHttp下载或者可以考虑同步的方式下载
## Retrofit配置
### 1.配置OkHttpClient
```java
 OkHttpClient okHttpClient = new OkHttpClient.Builder().addInterceptor(new Interceptor() {
            @Override
            public okhttp3.Response intercept(Chain chain) throws IOException {
                return null;
            }
        })
        
Retrofit retrofit = new RetrofitBuilder()
                .callFactory(okHttpClient)
                .build();
```
### 添加Converter
**添加Gson Converter**
Retrofit现在已经不提供默认的 converter 了，如果不显性的声明一个可用的 Converter 的话，Retrofit 是会报错
使用官方提供的Converter
添加依赖
```gradle
//使用Gson
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
//使用JackSon
compile 'com.squareup.retrofit2:converter-jackson:2.1.0'
```
在Retrofit2中可以绑定多个Converter对象，服务器返回不同格式的数据可以使用不同的Converter进行反序列化。并且Converter是按照被添加的顺序顺次调用的，当前Converter不能解析该类型的数据时，会调用下一个Converter。对于Proto格式的数据，Protobuff 都是从一个名叫 Message 或者 MessageLite 的类继承而来。所以，通过判断这个类是否继承自 Message，继而选择对应的Converter对象进行反序列化。但通常JSON格式的数据没有明确的判断条件，所有JSON Converter对象会处理任何的数据，所以一定要将JSON Converter对象放在最后
```java
//添加 converter 的顺序很重要
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(ProtoConverterFactory.create())
    .addConverterFactory(GsonConverterFactory.create())
    .build();
```
**自定义Gson**
当使用官方提供的 Gson Converter时，如果需要调整一些格式，例如时间格式，可以自定义Gson
```java
Gson gson = new Gson.Builder()
        .setDateFormat("yyyy-MM-dd'T'HH:mm:ssZ")
        .create();
        
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("https://api.github.com")
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build();
```
如果希望自定义Converter，需要分别创建抽象类Converter.Factory的子类，接口Converter<T, RequestBody>的实现，和接口Converter<ResponseBody, T>的实现，例如当希望使用FastJson解析数据时
定义FastJsonConverterFactory
```java
public final class FastJsonConverterFactory extends Converter.Factory {
    public static FastJsonConverterFactory create() {
        return create(new FastJsonConfig());
    }

    private static FastJsonConverterFactory create(FastJsonConfig fastJsonConfig) {
        return new FastJsonConverterFactory(fastJsonConfig);
    }
    
    private final FastJsonConfig fastJsonConfig;

    private FastJsonConverterFactory(FastJsonConfig fastJsonConfig) {
        if(fastJsonConfig == null) throw new NullPointerException("fastJsonConfig == null");
        this.fastJsonConfig = fastJsonConfig;
    }
    
    @Override
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit) {
        return new FastJsonResponseBodyConverter<>(fastJsonConfig,type);
    }

    @Override
    public Converter<?, RequestBody> requestBodyConverter(Type type, Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
        return new FastJsonRequestBodyConverter<>(fastJsonConfig);
    }
}
```
实现FastJsonRequestBodyConverter
```java
final class FastJsonRequestBodyConverter<T> implements Converter<T, RequestBody> {
    private static final MediaType MEDIA_TYPE = MediaType.parse("application/json; charset=UTF-8");
    private FastJsonConfig fastJsonConfig;
   
    public FastJsonRequestBodyConverter(FastJsonConfig fastJsonConfig) {
        this.fastJsonConfig = fastJsonConfig;
    }

    @Override
    public RequestBody convert(T value) throws IOException {
        SerializeConfig serializeConfig = fastJsonConfig.getSerializeConfig();
        SerializerFeature[] serializerFeatures = fastJsonConfig.getSerializerFeatures();

        byte[] content;
        if (serializeConfig != null) {
            if (serializerFeatures != null) {
                content = JSON.toJSONBytes(value, serializeConfig, serializerFeatures);
            } else {
                content = JSON.toJSONBytes(value, serializeConfig);
            }
        } else {
            if (serializerFeatures != null) {
                content = JSON.toJSONBytes(value, serializerFeatures);
            } else {
                content = JSON.toJSONBytes(value);
            }
        }
        return RequestBody.create(MEDIA_TYPE, content);
    }
}
```
实现FastJsonResponseBodyConverter
```java
final class FastJsonResponseBodyConverter<T> implements Converter<ResponseBody, T> {
    private Type type;
    private FastJsonConfig fastJsonConfig;
    public FastJsonResponseBodyConverter(FastJsonConfig fastJsonConfig, Type type) {
        this.fastJsonConfig = fastJsonConfig;
        this.type = type;
    }

    @Override
    public T convert(ResponseBody value) throws IOException {
        ParserConfig config = fastJsonConfig.getParserConfig();
        int featureValues = fastJsonConfig.getFeatureValues();
        Feature[] features = fastJsonConfig.getFeatures();
        //使用Okio提供效率
        BufferedSource bufferedSource = Okio.buffer(value.source());
        String tempStr = bufferedSource.readUtf8();
        bufferedSource.close();
        
        return JSON.parseObject(tempStr,type, config, featureValues,
                features != null ? features : FastJsonConfig.EMPTY_SERIALIZER_FEATURES);
        //另外一种写法
//        try {
//            return JSON.parseObject(value.string(), type, config, featureValues,
//                    features != null ? features : FastJsonConfig.EMPTY_SERIALIZER_FEATURES);
//        } finally {
//            value.close();
//        }
    }
}
```
实现FastJson，为FastJson提供配置
```java
public class FastJsonConfig {
    public static final Feature[] EMPTY_SERIALIZER_FEATURES = new Feature[0];


    private ParserConfig parserConfig;
    private int featureValues;
    private Feature[] features;

    private SerializeConfig serializeConfig;
    private SerializerFeature[] serializerFeatures;


    public FastJsonConfig() {
        //使用默认反序列化配置
        this.parserConfig = ParserConfig.getGlobalInstance();
        this.featureValues = JSON.DEFAULT_PARSER_FEATURE;
        
         //使用默认序列化配置
//        this.serializeConfig = SerializeConfig.globalInstance;
        //使用默认序列化配置
//        this.serializerFeatures = SerializerFeature.values();
        //使用自定义序列化配置
//        this.serializerFeatures = new SerializerFeature[] {
//                SerializerFeature.WriteNullBooleanAsFalse,//boolean为null时输出false
//                SerializerFeature.WriteMapNullValue, //输出空置的字段
//                SerializerFeature.WriteNonStringKeyAsString,//如果key不为String 则转换为String 比如Map的key为Integer
//                SerializerFeature.WriteNullListAsEmpty,//list为null时输出[]
//                SerializerFeature.WriteNullNumberAsZero,//number为null时输出0
//                SerializerFeature.WriteNullStringAsEmpty//String为null时输出""      
//        };
    }

    public ParserConfig getParserConfig() {
        return parserConfig;
    }
    
    public int getFeatureValues() {
        return featureValues;
    }
    
    public Feature[] getFeatures() {
        return features;
    }
    
    public SerializeConfig getSerializeConfig() {
        return serializeConfig;
    }

    public SerializerFeature[] getSerializerFeatures() {
        return serializerFeatures;
    }
}
```
### 增加日志信息
在retrofit2.0中是没有日志功能的。但是retrofit2.0中依赖OkHttp，所以也就能够通过OkHttp中的interceptor来实现实际的底层的请求和响应日志，为其自定自定义的OkHttpClient。
添加依赖：
```gradle
compile 'com.squareup.okhttp3:logging-interceptor:3.1.2'
```
创建Interceptor：
```java
HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
            loggingInterceptor.setLevel(BuildConfig.DEBUG?
                    HttpLoggingInterceptor.Level.BODY: HttpLoggingInterceptor.Level.NONE);
                    
OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .addInterceptor(interceptor)
                .build();
                
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(okHttpClient)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
```
### Retrofit设置缓存
当需要设置缓存时，需要对底层依赖的的OkHttp进行设置

**开启OkHttp缓存**
```java
  //设置缓存目录
 File httpCacheDirectory = new File(App.getInstance().getCacheDir(), "responses");
 int cacheSize = 10 * 1024 * 1024;
 Cache cache = new Cache(httpCacheDirectory, cacheSize);
 
 mOkHttpClient = new OkHttpClient.Builder()
                    .cache(cache)
                    .build();
```
**设置拦截器**
根据不同的缓存策略，可以创建不同的拦截器

缓存第一种类型
配置单个请求的@Headers，设置此请求的缓存策略,不影响其他请求的缓存策略,不设置则没有缓存。
```java
public interface Api {
    @Headers("Cache-Control:max-age=640000)
    Call<BaseModel<User>> getUser();
}
```

缓存第二种类型
有网和没网都先读缓存，统一缓存策略
```java
public class ForceCacheInterceptor  implements Interceptor {
    //缓存超时时间
    private static final int MAX_AGE = 60;

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        
        //读取请求头中配置的Cache-Control
        String cacheControl = request.cacheControl().toString();
        if (TextUtils.isEmpty(cacheControl)) {
            cacheControl = "public, max-age=" + MAX_AGE;
        }

        Response response = chain.proceed(request);

        //将缓存设置到响应中
        return response.newBuilder()
                .header("Cache-Control", cacheControl)
                .removeHeader("Pragma") //移除干扰信息
                .build();
    }
}

```
缓存第三种类型
结合前两种，离线读取本地缓存，在线获取最新数据(读取单个请求的请求头，亦可统一设置)
```java
public class OfflineCacheInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
    
        Request request = chain.request();

        if (!NetUtils.isConnected(App.getInstance())) {
            request = request.newBuilder()
                    .cacheControl(CacheControl.FORCE_CACHE)
                    .build();
        }

        Response response = chain.proceed(request);

        if (NetUtils.isConnected(App.getInstance())) {
            int maxAge = 0; // read from cache
            response = response.newBuilder()
                    .removeHeader("Pragma")// 清除头信息，因为服务器如果不支持，会返回一些干扰信息，不清除下面无法生效
                    .header("Cache-Control", "public ,max-age=" + maxAge)
                    .build();
        } else {
            int maxStale = 60 * 60 * 24 * 28; // tolerate 4-weeks stale
            response = response.newBuilder()
                    .removeHeader("Pragma")
                    .header("Cache-Control", "public, only-if-cached, max-stale=" + maxStale)
                    .build();
        }
        
        return response;
    }
}

```
### 异常处理
### 网络状态监听
### 设置公共参数
## Retrofit封装
## Retrofit源码解析