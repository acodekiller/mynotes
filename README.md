## 1.创建springboot项目：

### 1）改为阿里云镜像：http://start.aliyun.com

![image-20210226142647694](E:\资料文档\Typora\MyNotes\image-20210226142647694.png)

### 2）New project:

![image-20210226142854137](E:\资料文档\Typora\MyNotes\image-20210226142854137.png)

### 3）勾选所需开发工具等等

### 4）打开所需工具栏：

![20160624102544939](E:\资料文档\Typora\MyNotes\20160624102544939.jpg)

### 5）展开文件夹，方便操作：

![image-20210226143214911](E:\资料文档\Typora\MyNotes\image-20210226143214911.png)

### * 6）如果采用yml配置，需将默认的 applicaion.properties文件禁用（非常重要）：

![image-20210226143617049](E:\资料文档\Typora\MyNotes\image-20210226143617049.png)



# 2.开发要点：

### 1）用户验证部分：

#### i.步骤一，导入meavn依赖：

```xml
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.6.0</version>
</dependency>
```

#### ii.步骤二，编写配置文件：

![image-20210301162441307](E:\资料文档\Typora\MyNotes\image-20210301162441307.png)

```java
package com.lin.config.loginConfig;

import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ShiroConfig {
    @Bean
    public UserRealm userRealm() {
        return new UserRealm();
    }

    @Bean(name="securityManager")
    public DefaultWebSecurityManager getDefaultWebSecutiryManger(@Qualifier("userRealm") UserRealm userRealm){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);

        bean.setLoginUrl("/toLogin");

        return bean;
    }
}
```

```java
package com.lin.config.loginConfig;

import com.lin.pojo.UserInfo;
import com.lin.service.UserService;

import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

public class UserRealm extends AuthorizingRealm {

    @Autowired
    private UserService userService;

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        UsernamePasswordToken userToken = (UsernamePasswordToken) authenticationToken;
        UserInfo user = userService.queryByName(userToken.getUsername());
        if (user == null) {
            return null;
        }
        return new SimpleAuthenticationInfo(user, user.getUserpwd(), "");
    }

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

}

```

#### iii.通过数据库验证：

```java
@RequestMapping(value = "/index")
    public String login(@RequestParam String username, @RequestParam String password, Model model, HttpSession session){
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token =new UsernamePasswordToken(username,password);
        try{
            session.setAttribute("username",username);
            model.addAttribute("userInfo",username);
            subject.login(token);
            return "movie/basic-table3";
        }catch (UnknownAccountException e){
            model.addAttribute("msg","用户名错误");
            return "login/login";
        }catch (IncorrectCredentialsException e){
            model.addAttribute("msg","密码错误");
            return "login/login";
        }
    }
```



### 2）使用公共模板：

#### i.步骤一：

在templates文件下创建commons文件夹，再创建commons.html文件

#### ii.步骤二：

抽取公共部分，并添加标签`th:fragment="xxxx"`，使其他地方能够引用；

![image-20210227120024092](E:\资料文档\Typora\MyNotes\image-20210227120024092.png)

#### iii.步骤三：

在需要引用的地方添加`th:insert="~{xxx::xxx}"`标签即可。

![image-20210227120316376](E:\资料文档\Typora\MyNotes\image-20210227120316376.png)

#### iv.注意点：

部分网页在抽取标签后会使样式变坏，即使代码相同，在这种情况下，需要两者都引入需要的class样式。如：

![image-20210227120613871](E:\资料文档\Typora\MyNotes\image-20210227120613871.png)

（需引入公共部分的网页👆）

![image-20210227120736052](E:\资料文档\Typora\MyNotes\image-20210227120736052.png)

（被抽取出来的部分👆）



### 3）使用VUE获取后端数据：

#### a.导入vue、axios包：

```html
<script type="text/javascript" src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

#### b.编写后端数据接口：

![image-20210228125552909](E:\资料文档\Typora\MyNotes\image-20210228125552909.png)

#### c.编写script代码，引入后端数据：

![image-20210302165444712](E:\资料文档\Typora\MyNotes\image-20210302165444712.png)

![image-20210302165035892](E:\资料文档\Typora\MyNotes\image-20210302165035892.png)

#### d.使用后端数据：

![image-20210228125333274](E:\资料文档\Typora\MyNotes\image-20210228125333274.png)

#### e.解决vue闪烁问题：

（vue在未完成渲染前会出现`{{xxx}}`，使用此种方法能使在未完成渲染前不显示任何东西）

![image-20210228125857917](E:\资料文档\Typora\MyNotes\image-20210228125857917.png)

![image-20210228125928509](E:\资料文档\Typora\MyNotes\image-20210228125928509.png)

#### f.解决CORS跨域问题：

添加注解类：

```java
/**
 * 解决异步访问跨域
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        //本应用的所有方法都会去处理跨域请求
        registry.addMapping("/**")
                //允许远端访问的域名
                .allowedOrigins("http://localhost:8080")
                //允许请求的方法("POST", "GET", "PUT", "OPTIONS", "DELETE")
                .allowedMethods("*")
                //允许请求头
                .allowedHeaders("*");
    }
}
```

### 4）Ajax请求：

#### a.后端数据：

```java
@RestController
public class AjaxController {
    @Autowired
    private MovieService movieService;

    //获得一条电影记录
    @RequestMapping("/getOneMovie")
    public String getOneMovie(String username,Integer movieId) throws Exception{
        return new ObjectMapper().writeValueAsString(movieService.getOneMovie(username,movieId));
    }
}
```

#### b.前端获取两种方法

```html
<script>
    function changeFunction(value) {
        // $.ajax({
        //     url:"http://localhost:8080/getMovies",
        //     data:{"username":"15994941242","movieId":value},
        //     success:function (data) {
        //         console.log(data)
        //     }
        // })
        $.post("http://localhost:8080/getOneMovis",			//对应后端方法
        {
            username: userInfo,		//对应后端的username
            movieId:value			//对应后端的movieId
        },
        function(data){
            alert("数据: \n" + data + "\n状态: " + status);
        });
    }
</script>
```

### 5）删除前确认按钮

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
<button type="button" onclick="del()">删除</button>
</body>
<script>
    function del() {
    var msg = "您确定要删除该条电影信息吗？\n\n请确认！";
    if (confirm(msg)==true){
    return true;
    }else{
    return false;
    }
    }
</script>
</html>
```



# 3.用户登录邮箱验证码

### 项目结构

![image-20210325224010370](E:\资料文档\Typora\MyNotes\image-20210325224010370.png)

### 1）创建表user

![image-20210324133505450](E:\资料文档\Typora\MyNotes\image-20210324133505450.png)



### 2）导入依赖

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```



### 3）配置yml文件

```yaml
mybatis:
  type-aliases-package: com.lin.pojo
  mapper-locations: classpath:/mapper/*.xml
server:
  port: 8080

spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://8.140.22.199:3306/test?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
  thymeleaf:
    cache: false
  #邮箱配置
  mail:
    host: smtp.qq.com
    username: 1351503414@qq.com
    password: krlntvulffspfifd    #发送短信后它给你的授权码
    default-encoding: utf-8
    #在阿里云服务器上端口25需申请才能打开，将处理发送邮箱端口改为”465“
    port: 465
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true
          socketFactory:
            port:port: 465
            class: javax.net.ssl.SSLSocketFactory
            fallback: false

```



### 4）编写pojo层

User.class

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String username;
    private String password;
    private String email;
}

```



### 5）编写vo层

UserVo.class

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserVo {
    private String username;
    private String password;
    private String email;
    private String code;//验证码
}
```

UserVoToUser.class

```java
public class UserVoToUser {

    public static User toUser(UserVo userVo){
        User user = new User();
        user.setUsername(userVo.getUsername());
        user.setPassword(userVo.getPassword());
        user.setEmail(userVo.getEmail());
        return user;
    }

}
```



### 6）编写mapper层

UserMapper.class

```java
@Repository
@Mapper
public interface UserMapper {

    //查询所有用户
    List<User> queryAll();

    //插入一个用户
    void insertUser(User user);

}
```

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.lin.mapper.UserMapper">

    <select id="queryAll" resultType="user">
        SELECT * FROM test.user
    </select>

    <insert id="insertUser" parameterType="user">
        INSERT INTO test.user(username,password,email) VALUES (#{username},#{password},#{email})
    </insert>

</mapper>
```



### 7）编写services层

UserService.class

```java
public interface UserService {
    //查询所有用户
    List<User> queryAll();

    //插入一个用户
    void insertUser(User user);
}
```

UserServiceImpl.class

```java
@Service
public class UserServiceImpl implements UserService{

    @Autowired
    private UserMapper userMapper;

    @Override
    public List<User> queryAll() {
        return userMapper.queryAll();
    }

    @Override
    public void insertUser(User user) {
        userMapper.insertUser(user);
    }
}
```



### 8）定制邮件服务MailService

```java
package com.lin.services;

import com.lin.mapper.UserMapper;
import com.lin.pojo.User;
import com.lin.vo.UserVo;
import com.lin.vo.UserVoToUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;

import javax.mail.internet.MimeMessage;
import javax.servlet.http.HttpSession;
import java.util.Random;

@Service
public class MailService {

    @Autowired
    private JavaMailSender mailSender;

    @Autowired
    private UserMapper userMapper;

    @Value("1351503414@qq.com")
    private String from; //邮件发送人

    //发送验证码
    public boolean sendMail(String email, HttpSession httpSession) {
        try {
            MimeMessage mimeMessage = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
            //生成随机数
            String code = randomCode();
            helper.setSubject("验证码邮件");//主题
            helper.setText("<strong>尊敬的用户：您好！</strong><br>您正在进行<span style=\"color: red\">" +
                    "注册账号</span>操作，请在验证码输入框中输入：<span style=\"color:#f60;font-size: 24px\">" + code +
                    "</span>，以完成操作。" +
                    "<p style=\"color:#747474;\">\n" +
                    "注意：此操作可能会修改您的密码、登录邮箱或绑定手机。如非本人操作，请忽略此邮件\n" +
                    "<br>（工作人员不会向你索取此验证码，请勿泄漏！)\n" +
                    "</p>\n" +
                    "————————————————\n", true);
            helper.setTo(email);
            helper.setFrom(from);
            mailSender.send(mimeMessage);//发送
            httpSession.setMaxInactiveInterval(60);//设置60s过期
            httpSession.setAttribute("email",email);
            httpSession.setAttribute("code",code);

            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    //检验验证码
    public boolean registered(UserVo userVo, HttpSession httpSession){
        String email = (String)httpSession.getAttribute("email");
        String code = (String)httpSession.getAttribute("code");

        //获取表单填入的验证码
        String inputcode = userVo.getCode();
        //获取表单填入的邮箱
        String inputemail = userVo.getEmail();

        if(!inputemail.equals(email)){
            System.out.println(email);
            System.out.println(inputemail);
            return false;
        }else if(!code.equals(inputcode)){
            System.out.println("验证码错误！");
            return false;
        }
        User user = UserVoToUser.toUser(userVo);
        userMapper.insertUser(user);
        return true;
    }

    public String randomCode() {
        StringBuilder str = new StringBuilder();
        Random random = new Random();
        for (int i = 0; i < 6; i++) {
            str.append(random.nextInt(10));
        }
        return str.toString();
    }

}

```



### 9）编写contoller层

emailController.class

```java
@Controller
public class emailController {

    @Autowired
    private MailService mailService;

    @RequestMapping("/login")
    public String login(){
        return "register";
    }

    //注册账户
    @ResponseBody
    @RequestMapping("/addUser")
    public boolean addUser(UserVo userVo, HttpSession session){
        return mailService.registered(userVo,session);
    }

    //发送验证码
    @ResponseBody
    @RequestMapping("/sendMail")
    public boolean sendMail(String email,HttpSession session){
        mailService.sendMail(email,session);
        return true;
    }
}
```



### 10）注册页

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js"></script>
</head>
<body>
<div>
    <form action="/addUser" method="post">
    <span>
        账号：<input type="text" name="username"><br>
    </span>
        <span>
        密码：<input type="password" name="password"><br>
    </span>
        <span>
        邮箱：<input type="email" name="email" id="email"><br>
    </span>
        <span>
        验证码：<input type="text" name="code"><br>
    </span>
        <button type="button" id="getCode">获取验证码</button>
        <button type="submit">提交</button>
        <button type="reset">重置</button>
    </form>
</div>
</body>
<script>
    $('#getCode').click(function () {
        $.post("/sendMail",
            {
                email: $('#email').val()
            },
            function(data){
                console.log(data)
            }
        );
        var time = 60;
        $(this).attr("disabled", true);
        var timer = setInterval(function() {
            if (time == 0) {
                clearInterval(timer);
                $("#getCode").attr("disabled", false);
                $("#getCode").val("获取验证码");
                $("#getCode").removeClass("on");
            } else {
                $('#getCode').val(time + "秒");
                time--;
            }
        }, 1000);
    });
</script>

</html>
```





# 4.使用mybatis-generator工具

## 1）导入依赖

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.5</version>
</dependency>
```



## 2）导入插件

```xml
<!--引入mybatis生成器-->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
<!--引入mybatis生成器-->
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.5</version>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.22</version>
        </dependency>
    </dependencies>
</plugin>
```



## 3）在resource文件下新建generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--mysql 连接数据库jar 这里选择自己本地位置 这里的版本必须与pom.xml中的mysql-connector-java对应上 -->
    <classPathEntry location="C:\Users\Lin\.m2\repository\mysql\mysql-connector-java\8.0.22\mysql-connector-java-8.0.22.jar" />
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://39.97.179.241:3306/test?useSSL=false" userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和
           NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- targetProject:生成PO类的位置 -->
        <javaModelGenerator targetPackage="com.lin.pojo"
                            targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置
           如果maven工程只是单独的一个工程，targetProject="src/main/java"
           若果maven工程是分模块的工程，targetProject="所属模块的名称"，例如：
           targetProject="ecps-manager-mapper"，下同-->
        <sqlMapGenerator targetPackage="com.lin.mapper"
                         targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
        <!-- targetPackage：mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.lin.mapper"
                             targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>
        <!-- 指定数据库表 -->
        <table schema="flow" tableName="flow"></table>
<!--        <table schema="" tableName="success_killed"></table>-->
    </context>
</generatorConfiguration>
```



## 4）编写application.properties文件

```properties
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql://39.97.179.241:3306/test?useSSL=false&useUnicode=true&characterEncoding=utf8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
mybatis.type-aliases-package=com.lin.pojo
```



## 5）使用插件

![image-20210324194328850](E:\资料文档\Typora\MyNotes\image-20210324194328850.png)



## 6）效果

![image-20210324194420692](E:\资料文档\Typora\MyNotes\image-20210324194420692.png)



## 7）注意

此时FlowMapper文件在mapper包下（不在resource文件夹下），而properties文件无法自定扫描包下目录，可在pom.xml中的build下导入以下文件：

```xml
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
    </resource>
    <resource>
        <directory>src/main/resources</directory>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
    </resource>
</resources>
```







# 5.使用Redis做数据库缓存

### 1）添加依赖

```xml
<!--集成redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-redis</artifactId>
    <version>1.4.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>

```

### 2）编写application.properties

```properties
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=39.97.179.241
# Redis服务器连接端口
spring.redis.port=6371
# Redis服务器连接密码（默认为空）
spring.redis.password=123456
## 连接超时时间（毫秒）
spring.redis.timeout=30000
#连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
#连接池中的最小空闲连接
spring.redis.pool.min-idle=0
#连接池中的最大空闲连接
spring.redis.pool.max-idle=8


spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql://39.97.179.241:3306/test?useSSL=false&useUnicode=true&characterEncoding=utf8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
mybatis.type-aliases-package=com.lin.pojo
```



### 3）编写工具类

#### RedisUtil.class

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.*;
import org.springframework.stereotype.Service;

import java.io.Serializable;
import java.util.List;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Service
public class RedisUtils {
    @Autowired
    private RedisTemplate redisTemplate;
    /**
     * 写入缓存
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
    /**
     * 写入缓存设置时效时间
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value, Long expireTime , TimeUnit timeUnit) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            redisTemplate.expire(key, expireTime, timeUnit);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
    /**
     * 批量删除对应的value
     * @param keys
     */
    public void remove(final String... keys) {
        for (String key : keys) {
            remove(key);
        }
    }
    /**
     * 批量删除key
     * @param pattern
     */
    public void removePattern(final String pattern) {
        Set<Serializable> keys = redisTemplate.keys(pattern);
        if (keys.size() > 0){
            redisTemplate.delete(keys);
        }
    }
    /**
     * 删除对应的value
     * @param key
     */
    public void remove(final String key) {
        if (exists(key)) {
            redisTemplate.delete(key);
        }
    }
    /**
     * 判断缓存中是否有对应的value
     * @param key
     * @return
     */
    public boolean exists(final String key) {
        return redisTemplate.hasKey(key);
    }
    /**
     * 读取缓存
     * @param key
     * @return
     */
    public Object get(final String key) {
        Object result = null;
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        result = operations.get(key);
        return result;
    }
    /**
     * 哈希 添加
     * @param key
     * @param hashKey
     * @param value
     */
    public void hmSet(String key, Object hashKey, Object value){
        HashOperations<String, Object, Object> hash = redisTemplate.opsForHash();
        hash.put(key,hashKey,value);
    }
    /**
     * 哈希获取数据
     * @param key
     * @param hashKey
     * @return
     */
    public Object hmGet(String key, Object hashKey){
        HashOperations<String, Object, Object>  hash = redisTemplate.opsForHash();
        return hash.get(key,hashKey);
    }
    /**
     * 列表添加
     * @param k
     * @param v
     */
    public void lPush(String k,Object v){
        ListOperations<String, Object> list = redisTemplate.opsForList();
        list.rightPush(k,v);
    }
    /**
     * 列表获取
     * @param k
     * @param l
     * @param l1
     * @return
     */
    public List<Object> lRange(String k, long l, long l1){
        ListOperations<String, Object> list = redisTemplate.opsForList();
        return list.range(k,l,l1);
    }
    /**
     * 集合添加
     * @param key
     * @param value
     */
    public void add(String key,Object value){
        SetOperations<String, Object> set = redisTemplate.opsForSet();
        set.add(key,value);
    }
    /**
     * 集合获取
     * @param key
     * @return
     */
    public Set<Object> setMembers(String key){
        SetOperations<String, Object> set = redisTemplate.opsForSet();
        return set.members(key);
    }
    /**
     * 有序集合添加
     * @param key
     * @param value
     * @param scoure
     */
    public void zAdd(String key,Object value,double scoure){
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        zset.add(key,value,scoure);
    }
    /**
     * 有序集合获取
     * @param key
     * @param scoure
     * @param scoure1
     * @return
     */
    public Set<Object> rangeByScore(String key,double scoure,double scoure1){
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        return zset.rangeByScore(key, scoure, scoure1);
    }
}
```

#### RedisConfig.class

```java
import java.lang.reflect.Method;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.databind.ObjectMapper;

@Configuration
@EnableCaching
public class RedisConfig {
    @Value("${spring.redis.host}")
    private String host;
    @Value("${spring.redis.port}")
    private int port;
    @Value("${spring.redis.timeout}")
    private int timeout;
    @Value("${spring.redis.password}")
    private String password;
    @Value("${spring.redis.pool.max-active}")
    private int maxActive;
    @Value("${spring.redis.pool.max-idle}")
    private int maxIdle;
    @Value("${spring.redis.pool.min-idle}")
    private int minIdle;

    @Bean
    public KeyGenerator wiselyKeyGenerator(){
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    @Bean
    public JedisConnectionFactory redisConnectionFactory() {
        JedisConnectionFactory factory = new JedisConnectionFactory();
        factory.setHostName(host);
        factory.setPort(port);
        factory.setTimeout(timeout); //设置连接超时时间
        factory.setPassword(password);
        factory.getPoolConfig().setMaxIdle(maxIdle);
        factory.getPoolConfig().setMinIdle(minIdle);
        factory.getPoolConfig().setMaxTotal(maxActive);
        return factory;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        setSerializer(template); //设置序列化工具，这样ReportBean不需要实现Serializable接口
        template.afterPropertiesSet();
        return template;
    }

    private void setSerializer(StringRedisTemplate template) {
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
    }

}
```





### 4）编写pojo层

#### Flow.class

```java
@Data
public class Flow {
    private String id;

    private String flowNum;

    private String orderNum;

    private String productId;

    private String paidAmount;

    private Integer paidMethod;

    private Integer buyCounts;

    private Date createTime;
    
}
```



### 5）编写mapper层

#### FlowMapper.class

```java
@Mapper
@Component
public interface FlowMapper {
    long countByExample(FlowExample example);

    int deleteByExample(FlowExample example);

    int deleteByPrimaryKey(String id);

    int insert(Flow record);

    int insertSelective(Flow record);

    List<Flow> selectByExample(FlowExample example);

    Flow selectByPrimaryKey(String id);

    int updateByExampleSelective(@Param("record") Flow record, @Param("example") FlowExample example);

    int updateByExample(@Param("record") Flow record, @Param("example") FlowExample example);

    int updateByPrimaryKeySelective(Flow record);

    int updateByPrimaryKey(Flow record);
}
```

FlowMappper.xml文件（略）



### 6）编写service层

#### FlowService.class

```java
public interface FlowService {

    long countByExample(FlowExample example);

    int deleteByExample(FlowExample example);

    int deleteByPrimaryKey(String id);

    int insert(Flow record);

    int insertSelective(Flow record);

    List<Flow> selectByExample(FlowExample example);

    Flow selectByPrimaryKey(String id);

    int updateByExampleSelective(@Param("record") Flow record, @Param("example") FlowExample example);

    int updateByExample(@Param("record") Flow record, @Param("example") FlowExample example);

    int updateByPrimaryKeySelective(Flow record);

    int updateByPrimaryKey(Flow record);

}
```

#### FlowServiceImpl.class

```java
@Service
public class FlowServiceImpl implements FlowService{

    @Autowired
    private FlowMapper flowMapper;

    @Override
    public long countByExample(FlowExample example) {
        return flowMapper.countByExample(example);
    }

    @Override
    public int deleteByExample(FlowExample example) {
        return flowMapper.deleteByExample(example);
    }

    @Override
    public int deleteByPrimaryKey(String id) {
        return flowMapper.deleteByPrimaryKey(id);
    }

    @Override
    public int insert(Flow record) {
        return flowMapper.insert(record);
    }

    @Override
    public int insertSelective(Flow record) {
        return flowMapper.insertSelective(record);
    }

    @Override
    public List<Flow> selectByExample(FlowExample example) {
        return flowMapper.selectByExample(example);
    }

    @Override
    public Flow selectByPrimaryKey(String id) {
        return flowMapper.selectByPrimaryKey(id);
    }

    @Override
    public int updateByExampleSelective(Flow record, FlowExample example) {
        return flowMapper.updateByExampleSelective(record,example);
    }

    @Override
    public int updateByExample(Flow record, FlowExample example) {
        return flowMapper.updateByExample(record,example);
    }

    @Override
    public int updateByPrimaryKeySelective(Flow record) {
        return flowMapper.updateByPrimaryKeySelective(record);
    }

    @Override
    public int updateByPrimaryKey(Flow record) {
        return flowMapper.updateByPrimaryKey(record);
    }
}
```



### 7）编写Controller层

#### RedisController

```java
@RestController
public class RedisController {
    public static final Logger log = LoggerFactory.getLogger(RedisController.class);

    @Autowired
    private FlowService flowService;

    @Autowired
    private RedisUtils redisUtils;

    @RequestMapping("/selectByPrimaryKey")
    public String selectByPrimaryKey(String id) {
        //查询缓存中是否存在
        boolean hasKey = redisUtils.exists(id);
        String str = "";
        if (hasKey) {
            //获取缓存
            Object object = redisUtils.get(id);
            log.info("从缓存获取的数据" + object);
            str = object.toString();
        } else {
            //从数据库中获取信息
            log.info("从数据库中获取数据");
            str = flowService.selectByPrimaryKey(id).toString();
            //数据插入缓存（set中的参数含义：key值，user对象，缓存存在时间10（long类型），时间单位）
            redisUtils.set(id, str, 10L, TimeUnit.MINUTES);
            log.info("数据插入缓存" + str);
        }
        return str;
    }


    public static void main(String[] args){
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setContentType(MediaType.APPLICATION_JSON);

        RestTemplate restTemplate = new RestTemplate();
        HttpEntity<String > entity = new HttpEntity<>("a",httpHeaders);
        ResponseEntity responseEntity = restTemplate.postForEntity("http://localhost:8080/selectByPrimaryKey?id=1",entity,String.class);
        System.out.println(responseEntity.getBody());
    }

}
```



## 6.支付宝支付功能