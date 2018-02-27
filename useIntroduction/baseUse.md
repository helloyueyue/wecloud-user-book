# 基本用法
本地开发模式勾选右侧profiles中的local和wdnexus两项,如图：![](/assets/img4.png)
local为本地开发模式，项目运行自动读取配置文件application.properties。
wdnexus为maven私服配置,勾选后将从私服库https://wdnexus.mapbar.com/content/groups/public/下下载wecloud框架。
项目打包部署的时候需要勾选sandbox和wdnexus选项，选择sandbox项进行打包则不会将配置文件打包进去，这样方便以后动态修改配置文件。
## 搭建简单的SpringBoot项目
建立基本的springboot项目结构,在Application中添加注解自动扫描包加载配置文件，基础的Application启动类示例如下：
```
@EnableAutoConfiguration
@Configuration
@ComponentScan(basePackages = {"com.navinfo.wecloud.demo"})
public class Application {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.setWebEnvironment(true);
        app.run(args);
    }


}
```
右键Application类启动项目，启动成功如下：
![](/assets/img5.png)
这是最基本的SpringBoot项目，默认启动端口为8080。
浏览器中访问效果：
![](/assets/img6.png)
## 访问静态页面
在resources下新建配置文件application.properties
配置服务端口号 server.port=1111
在resources下创建目录static,在该目录下创建index.html。
如图：
![](/assets/img7.png)
![](/assets/img8.png)
再次启动Application类，在浏览器上访问效果如下：
![](/assets/img9.png)
## 编写rest接口
下面我们来写一个rest接口：
在controller目录下新建ApiRequestController类,在该类下定义接口，跟SpringMVC框架差不多一样的用法。
接口返回可以使用Wecloud中封装的通用类：
com.navinfo.wecloud.common.rest.response.CommonResult
您可以这样使用来返回不同的状态码和提示信息以及data数据体：
```
CommonResult commonResult=new CommonResult();
Object result =…
commonResult.setCode(200);
commonResult.setMessage("请求成功！");
commonResult.setData(result);
```
在get和post请求中我们可以分别使用@RequestParam和 @RequestBody来传参，其中@RequestBody可以传一个自定义的对象，你可以新建一个请求实体类。如下：
![](/assets/img10.png)
![](/assets/img11.png)
## RestTemplate使用
下面我们来介绍一下RestTemplate的使用：
在项目中调用第三方的http接口我们常常使用RestTemplate。
在Application启动类中注册Bean.
```
@Bean
public RestTemplate restTemplate(){
    HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
    httpRequestFactory.setConnectionRequestTimeout(10000);
    httpRequestFactory.setConnectTimeout(10000);
    httpRequestFactory.setReadTimeout(10000);

    return new RestTemplate(httpRequestFactory);
}
```
然后就可以在项目中这样使用RestTemplate:
```
@Autowired
private RestTemplate restTemplate;
```
RestTemplate的Get请求用法：
```
String result = restTemplate.getForObject(testUrl1+"?location="+location,String.class);
```

RestTemplate的Post请求用法1（传Map）：
```
Map<String, String> requestParam = new HashMap<>();
requestParam.put("sid", sid);
String result = restTemplate.postForObject(testUrl2,requestParam,String.class);
```
RestTemplate的Post请求用法2（传对象）：
```
PostRequestResult result = restTemplate.postForObject(testUrl2,testPostRequest, PostRequestResult.class);
```




