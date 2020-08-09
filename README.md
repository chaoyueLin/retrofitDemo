# retrofitDemo
## RESTful API

在正式开始了解Retrofit的使用之前，我们有必要先了解一个概念，即RESTful。这是因为Retrofit的初衷就是根据RESTful风格的API来进行封装的。
对RESTful的核心思想做一个最简练的总结，那就是：所谓”资源”，就是网络上的一个实体，或者说是网络上的一个具体信息。那么我们访问API的本质就是与网络上的某个资源进行互动而已。那么，因为我们实质是访问资源，所以RESTful设计思想的提出者Fielding认为：
URI当中不应当出现动词，因为”资源“表示一种实体，所以应该用名词表示，而动词则应该放在HTTP协议当中。那么举个最简单的例子：
> 
xxx.com/api/createUser
xxx.com/api/getUser
xxx.com/api/updateUser
xxx.com/api/deleteUser


这样的API风格我们应该很熟悉，但如果要遵循RESTful的设计思想，那么它们就应该变为类似下面这样：

> 
[POST]xxx.com/api/User
[GET] xxx.com/api/User
[PUT]xxx.com/api/User
[DELETE]xxx.com/api/User



也就是说：因为这四个API都是访问服务器的USER表，所以在RESTful里URL是相同的，而是通过HTTP不同的RequestMethod来区分增删改查的行为。
而有的时候，如果某个API的行为不好用请求方法描述呢？比如说，A向B转账500元。那么，可能会出现如下设计：

> 
POST /accounts/1/transfer/500/to/2

在RESTful的理念里，如果某些动作是HTTP动词表示不了的，你就应该把动作做成一种资源。可以像下面这样使用它：

> 
　　POST /transaction HTTP/1.1
　　Host: 127.0.0.1
　　from=1&to=2&amount=500.00


好了，当然实际来说RESTful肯定不是就这点内容。这里我们只是了解一下RESTful最基本和核心的设计理念。

## 简介
[Retrofit](http://square.github.io/retrofit/) 是 Square 推出的 HTTP 框架，主要用于 Android 和 Java。Retrofit 将网络请求变成方法的调用，使用起来非常简洁方便。

### 模型
retrofit模型如下：
![](http://upload-images.jianshu.io/upload_images/1638147-7267143bb6a3076b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

retrofit模型
> 
1) POJO或模型实体类 : 从服务器获取的JSON数据将被填充到这种类的实例中。
2) 接口 : 我们需要创建一个接口来管理像GET,POST...等请求的URL，这是一个服务类。
3) RestAdapter类 : 这是一个REST客户端(RestClient)类，retrofit中默认用的是Gson来解析JSON数据，你也可以设置自己的JSON解析器

## 使用
* 创建一个Retrofit 对象（核心用法一）
		Retrofit retrofit = new Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())//解析方法
             //这里建议：- Base URL: 总是以/结尾；- @Url: 不要以/开头
            .baseUrl("http://102.10.10.132/api/")
            .build();
* 接口申明（核心用法二）

	按照一个http请求对应的写法
	http://102.10.10.132/api/News/1
	http://102.10.10.132/api/News/{资讯id}

        public interface NewsService {
          /**
           * 根据newsid获取对应的资讯数据
           * 如果不需要转换成Json数据,可以用了ResponseBody;
           * @param newsId
           * @return call
           */
          @GET("News/{newsId}")
          Call<News> getNews(@Path("newsId") String newsId);
        }
    或
    http://102.10.10.132/api/News/1/类型1
    http://102.10.10.132/api/News/{资讯id}/{类型}

        @GET("News/{newsId}/{type}")
        Call<NewsBean> getItem(@Path("newsId") String newsId， @Path("type") String type);
    样式3（参数在URL问号之后）

    http://102.10.10.132/api/News?newsId=1
    http://102.10.10.132/api/News?newsId={资讯id}

        @GET("News")
        Call<NewsBean> getItem(@Query("newsId") String newsId);
    或
    http://102.10.10.132/api/News?newsId=1&type=类型1
    http://102.10.10.132/api/News?newsId={资讯id}&type={类型}

        @GET("News")
        Call<NewsBean> getItem(@Query("newsId") String newsId， @Query("type") String type);
    样式4（多个参数在URL问号之后，且个数不确定）

    http://102.10.10.132/api/News?newsId=1&type=类型1...
    http://102.10.10.132/api/News?newsId={资讯id}&type={类型}...

        @GET("News")
        Call<NewsBean> getItem(@QueryMap Map<String, String> map);
    也可以

        @GET("News")
        Call<NewsBean> getItem(
                  @Query("newsId") String newsId，
                  @QueryMap Map<String, String> map);
    POST

    样式1（需要补全URL，post的数据只有一条reason）

    http://102.10.10.132/api/Comments/1
    http://102.10.10.132/api/Comments/{newsId}

        @FormUrlEncoded
        @POST("Comments/{newsId}")
        Call<Comment> reportComment(
            @Path("newsId") String commentId,
            @Field("reason") String reason);
    样式2（需要补全URL，问号后加入access_token，post的数据只有一条reason）

    http://102.10.10.132/api/Comments/1?access_token=1234123
    http://102.10.10.132/api/Comments/{newsId}?access_token={access_token}

        @FormUrlEncoded
        @POST("Comments/{newsId}")
        Call<Comment> reportComment(
            @Path("newsId") String commentId,
            @Query("access_token") String access_token,
            @Field("reason") String reason);
    样式3（需要补全URL，问号后加入access_token，post一个body（对象））

    http://102.10.10.132/api/Comments/1?access_token=1234123
    http://102.10.10.132/api/Comments/{newsId}?access_token={access_token}

        @POST("Comments/{newsId}")
        Call<Comment> reportComment(
            @Path("newsId") String commentId,
            @Query("access_token") String access_token,
            @Body CommentBean bean);

    上传文件的
        public interface FileUploadService {
            @Multipart
            @POST("upload")
            Call<ResponseBody> upload(@Part("description") RequestBody description,
                                      @Part MultipartBody.Part file);
        }

    如果你需要上传文件，和我们前面的做法类似，定义一个接口方法，需要注意的是，这个方法不再有 @FormUrlEncoded 这个注解，而换成了 @Multipart，后面只需要在参数中增加 Part 就可以了。

    	//构建要上传的文件
        File file = new File(filename);
        RequestBody requestFile =
                RequestBody.create(MediaType.parse("application/otcet-stream"), file);

        MultipartBody.Part body =
                MultipartBody.Part.createFormData("aFile", file.getName(), requestFile);

        String descriptionString = "This is a description";
        RequestBody description =
                RequestBody.create(
                        MediaType.parse("multipart/form-data"), descriptionString);
    DELETE

    样式1（需要补全URL）

    http://102.10.10.132/api/Comments/1
    http://102.10.10.132/api/Comments/{newsId}
    {access_token}

        @DELETE("Comments/{commentId}")
        Call<ResponseBody> deleteNewsCommentFromAccount(
            @Path("commentId") String commentId);
    样式2（需要补全URL，问号后加入access_token）

    http://102.10.10.132/api/Comments/1?access_token=1234123
    http://102.10.10.132/api/Comments/{newsId}?access_token={access_token}

        @DELETE("Comments/{commentId}")
        Call<ResponseBody> deleteNewsCommentFromAccount(
            @Path("accountId") String accountId，
            @Query("access_token") String access_token);
    PUT（这个请求很少用到，例子就写一个）

    http://102.10.10.132/api/Accounts/1
    http://102.10.10.132/api/Accounts/{accountId}

        @PUT("Accounts/{accountId}")
        Call<ExtrasBean> updateExtras(
            @Path("accountId") String accountId,
            @Query("access_token") String access_token,
            @Body ExtrasBean bean);



* 创建访问API的请求（核心用法三）
        NewsService api = retrofit.create(NewsService .class);
        Call<News> call = service.getNews("123456");
* 同步调用(核心用法四)
        News news = call.execute();
* 异步调用（核心用法五）
        call.enqueue(new Callback<News>(){
                 @Override
                 public void onResponse(Response<News> response) {
                     //成功返回数据后在这里处理，使用response.body();获取得到的结果
                     News news = response.body();
                  }
                  @Override
                  public voidonFailure(Throwable t) {
                     //请求失败在这里处理
                 }
             });
* 取消请求（核心用法六）
        call.cancel();
完成以上步骤就可以实现一个简单的网络请求了。

### Retrofit注解

方法注解，包含@GET、@POST、@PUT、@DELETE、@PATH、@HEAD、@OPTIONS、@HTTP。
标记注解，包含@FormUrlEncoded、@Multipart、@Streaming。
参数注解，包含@Query,@QueryMap、@Body、@Field，@FieldMap、@Part，@PartMap。
其他注解，@Path、@Header,@Headers、@Url


* @HTTP：可以替代其他方法的任意一种

       /**
         * method 表示请的方法，不区分大小写
         * path表示路径
         * hasBody表示是否有请求体
         */
        @HTTP(method = "get", path = "users/{user}", hasBody = false)
        Call<ResponseBody> getFirstBlog(@Path("user") String user);
* @Url：使用全路径复写baseUrl，适用于非统一baseUrl的场景。

        @GET
        Call<ResponseBody> v3(@Url String url);

* @Streaming:用于下载大文件

        @Streaming
        @GET
        Call<ResponseBody> downloadFileWithDynamicUrlAsync(@Url String fileUrl);
        ResponseBody body = response.body();
        long fileSize = body.contentLength();
        InputStream inputStream = body.byteStream();



* 动态设置Header值
        @GET("user")
        Call<User> getUser(@Header("Authorization") String authorization)
等同于 :

        //静态设置Header值
        @Headers("Authorization: authorization")//这里authorization就是上面方法里传进来变量的值
        @GET("widget/list")
        Call<User> getUser()

* @Headers 用于修饰方法,用于设置多个Header值：

        @Headers({
            "Accept: application/vnd.github.v3.full+json",
            "User-Agent: Retrofit-Sample-App"
        })
        @GET("users/{username}")
        Call<User> getUser(@Path("username") String username);
