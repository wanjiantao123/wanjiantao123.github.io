###生成代码的使用目的：

![生成代码的设置界面](/images/生成代码的设置界面.png)
生成的是mooc数据库中的coursevideo表，这个表是我为我正在做的教育网站设计的，以便于我能够在上传多个视频文件的时候存储我的视频名称和地址。



在将生成的代码使用在我的项目中，遇到了如下的错误，现记录如下：

一、在刚开始导入项目时会发现诸多报错，但大部分是包的路径不匹配，但也有不是这个原因，其中一个如下：
import com.github.pagehelper.PageHelper;报错。	
**错误原因**：仔细分析发现这并不是导入的包路径报错，而是没有导入Page Helper的依赖。
**解决方法**：在pom.xm中添加
![添加依赖的截图](/images/添加依赖.png)
二、CoursevideoServiceImpl这是一个Coursevideoservice实现类：
![add方法中报错](/images/add方法中报错.png)
![delete方法中报错](/images/delete方法中报错.png)
**错误原因**：如上是两个方法中的报错，那么错误信息是非常的相似，第一个是ResourUtil这个工具类返回的是String类型，而setId的参数类型是integer类型，类型不匹配，第二个是idlist是list<String>类型，而batchDelete的参数是list<Integer>类型
**解决方法**：因为他提供的这两条语句我都用不上，所以直接注释就好了。



三、在修改完导入代码的各种错误后启动项目：
启动项目时报错为：找不到CoursevideoDao这个mapper。错误原因：
![启动类截图](/images/启动类截图.png)

**错误原因**：我的启动类中，mapperscan注解原先是只有com.mooc.mapper。而我将coursevideoDao放入的是com.mooc.biz.dao这个包中，因此在扫描时扫描不到这个mapper。**解决方法如上图**，因为转移我嫌麻烦，所以只需要在原先的路径后再加上这个接口存放的路径



四、在使用原本的add功能，用来给我的coursevideo表添加数据时我是这样写的：

看似没有错误，实际在启动时会报空指针异常。
**错误原因如下图**：
必须在所有使用了dao的地方，进行@Autowired
![Autowired注解的使用](/images/Autowired注解的使用.png)

**解决方法1**：

![controller类下添加](/images/controller类下添加.png)
图 6 controller类下添加
![方法中添加](/images/方法中添加.png)
图 7 方法中添加

**解决方法2**：
![controller类下添加](/images/controller类下添加.png)
图 8 controller类下添加
![方法中](/images/方法中.png)
图 9方法中





五、我发现了在自动生成的代码中，并没有一个符合我当下所需要的功能的方法，它的查询只有分页查询和条件查询，因此我添加了它的两个查询功能：
![增加的方法](/images/增加的方法.png)
图 10 增加的方法

CoursevideoSql.Xml:
![xml中的sql语句](/images/xml中的sql语句.png)
图 11 xml中的sql语句

Controller中的使用（从上个jsp页面中接收courseid的值进行查询）：
![方法的应用](/images/方法的应用.png)
图 12方法的应用

效果图:
![效果图](/images/效果图.png)
图 13 效果图
Coursevideo表：
![coursevideo表截图](/images/coursevideo表截图.png)
图 14 coursevideo表截图

结尾：暂时的调试记录挑选了以上几个，生成的代码还在使用中，我抛弃了它生成的js、html的代码（这些代码的功能过于单一，用不上），目前写的教育网站项目可以正常跑通生成的代码，调试记录还会继续记录。