# ServiceFramework Wiki

[README-EN](https://github.com/allwefantasy/ServiceFramework/blob/master/README-EN.md)

ServcieFramework 定位在 **移动互联网后端** 领域,强调开发的高效性，其开发效率可以比肩Rails.

### 在Maven中使用该项目

在你的pom.xml 文件中中添加如下引用:

        <dependency>
            <groupId>net.csdn</groupId>
            <artifactId>ServiceFramework</artifactId>
            <version>1.0</version>
        </dependency>



接着确保 项目根目录下有config/application.yml,config/logging.yml 两个文件即可。示例可参看该项目中config文件夹。

1. ActiveRecord化的Model层，支持 MongoDB 和 MySQL.
  
  
		    List<Tag> tags = Tag.where(map("name","java")).fetch;
   
2. 完全重新设计的Controller层,大量便利的函数。创新的过滤器设计，比如下面的代码表示validate 方法会拦截 push方法

           static {
             beforeFilter("validate", WowCollections.map(only, WowCollections.list("push")));
           }

3. 大部分对象使用IOC自动管理,使用简单。
  
		   @inject
		   Service service;
   
4. 不依赖容器，单元测试简单，从action到service,都可做到测试代码最少
  
	     @Test
	     public void search() throws Exception {
	         RestResponse response = get("/doc/blog/search", map(
	                 "tagNames", "_10,_9"
	         ));
	         Assert.assertTrue(response.status() == 200);
	         Page page = (Page) response.originContent();
	         Assert.assertTrue(page.getResult().size() > 0);
	     }

5. Thrift 和 RESTFul 只需简单配置即可同时提供 Thrift 和 RESTFul 接口
    
			 
		###############http config##################
		http:
		    port: 7700
		    disable: false

		thrift:
		    disable: false
		    services:
		        net_csdn_controller_thrift_impl_CLoadServiceImpl:
		           port: 7701
		           min_threads: 100
		           max_threads: 1000		        

		    servers:
		        load: ["127.0.0.1:7701"]

	  
6. 支持 Velocity, 页面可直接访问所有实例变量以及helper类的方法。支持Velocity 进行模板配置

	 
			    @At(path = "/hello", types = GET)
			    public void hello() {
			        render(200, map(
			                "name", "ServiceFramework"
			        ), ViewType.html);
			    }  


## QuickStart

Step 1 >   克隆项目
 
 

	git clone https://github.com/allwefantasy/ServiceFramework
 
 
Step 2 >   导入到IDE.
 
Step 3 >   根据你自己的数据库信息 编辑修改 config/application.yaml .注意如果你使用mysql,需要disable 调 mongodb.反之亦然
  				
    datasources:
        mysql:
           host: 127.0.0.1
           port: 3306
           database: wow
           username: root
           password: root
           disable: false
        mongodb:
           host: 127.0.0.1
           port: 27017
           database: wow
           disable: false
        redis:
            host: 127.0.0.1
            port: 6379
            disable: true 		          
 
Step4 >   在Mysql中导入 sql/wow.sql.
 
Step5 >   新建 com.example.model.Tag 类.

			public class Tag extends Model 
			{
			
			}

Step6 >   新建 com.example.controller.http.TagController

          public class TagController extends ApplicationController 
			{
			   @At(path = "/hello", types = RestRequest.Method.GET)
			    public void hello() {
			        Tag tag = Tag.create(map("name", "java"));
			        tag.save();
			        render(200, map(
			                "tag", tag
			        ), ViewType.html);
			    }
			}
			
Step7 >	新建 template/tag/hello.vm


			Hello $tag.name!  Hello  world!		

Step8 >   创建启动类

    public class ExampleApplication {

    public static void main(String[] args) {
        ServiceFramwork.scanService.setLoader(ExampleApplication.class);
        Application.main(args);
    }
    }
    
Step9 >   运行  ExampleApplication

Step10 >  浏览器中输入  http://127.0.0.1:9002/hello .同时查看数据库，你会发现tag表已经有数据了。  


Step11 >  写个Action单元测试  编辑 runner.DynamicSuite  在 initEnv方法第一行处添加

      ServiceFramwork.scanService.setLoader(ExampleApplication.class);

Step12 > 创建测试类 test.com.example.TagControllerTest

    public class TagControllerTest extends BaseControllerTest {
	    @Test
	    public void testHello() throws Exception {
	        Tag.deleteAll();
	        RestResponse response = get("/hello", map());
	        Assert.assertTrue(response.status() == 200);
	        String result = response.content();
	        Assert.assertEquals("Hello java!  Hello  world!", result);
	    }
    }

Step13 >  运行 DynamicSuiteRunner 跑起测试

Step14 >  补充：你也可以不使用DynamicSuiteRunner去跑。直接使用IDE跑单元测试类。需要做的是在你的单元测试类中加几句代码：

    static {
        initEnv(ExampleApplication.class);
    }

加这句主要是保证启动容器，并且采用了合适的类加载器。

QuickStart 的一些常见错误:

1. application 文件 数据连接配置错误。单元测试一定需要单独配置test的配置。因为单元测试一般可能会会有数据清理等，系统强制使用
   test的配置。

2. ServiceFramework 是使用配置文件来找类并且加载的，所以你需要正确配置contorller等所在位置。在上述测试中，包名和类名必须保证和示例一致。如果你需要使用不同的package,那么你需要修改application.yml中的application 配置。如下:
  
		  application:
		    controller: com.example.controller.http
		    model:      com.example.model
		    document:   com.example.document
		    service:    com.example.service
		    util:       com.example.util
		    test:       test.com.example






Model层基于如下开源项目:
 
* [ActiveORM](https://github.com/allwefantasy/active_orm)
* [MongoMongo](https://github.com/allwefantasy/mongomongo)


ServiceFramework 不适合遗留项目。我们倾向于在一个全新的项目中使用它。

   
## Doc Links

* [Summary](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-start.md)
* [Model](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-model.md)
* [Controller](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-controller.md)
* [Test](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-test.md)

## Step by Step tutorial
Step-by-Step-tutorial-for-ServiceFramework(continue...)

* [Step-by-Step-tutorial-for-ServiceFramework(1)](https://github.com/allwefantasy/service_framework_example/blob/master/README.md)
* [Step-by-Step-tutorial-for-ServiceFramework(2)](https://github.com/allwefantasy/service_framework_example/blob/master/doc/Step-by-Step-tutorial-for-ServiceFramework\(2\).md)
* [Step-by-Step-tutorial-for-ServiceFramework(3)](https://github.com/allwefantasy/service_framework_example/blob/master/doc/Step-by-Step-tutorial-for-ServiceFramework\(3\).md)
* [Step-by-Step-tutorial-for-ServiceFramework(4)](https://github.com/allwefantasy/service_framework_example/blob/master/doc/Step-by-Step-tutorial-for-ServiceFramework\(4\).md)


##  Some projects based on ServiceFramework

* [QuickSand](https://github.com/allwefantasy/QuickSand)








