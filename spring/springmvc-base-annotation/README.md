# springmvc基础（基于注解）

## 目录<br/>
<a href="#一搭建hello-spring工程">一、搭建hello spring工程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#11-项目搭建">1.1 项目搭建</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#12-相关注解说明">1.2 相关注解说明</a><br/>
<a href="#二配置自定义拦截器">二、配置自定义拦截器</a><br/>
<a href="#三全局异常处理">三、全局异常处理 </a><br/>
<a href="#四参数绑定">四、参数绑定</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#41-参数绑定">4.1 参数绑定</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#42-关于日期格式转换的三种方法">4.2 关于日期格式转换的三种方法</a><br/>
<a href="#五数据校验">五、数据校验</a><br/>
<a href="#六文件上传与下载">六、文件上传与下载</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#61-文件上传">6.1 文件上传</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#62-文件下载">6.2 文件下载</a><br/>
<a href="#七Restful风格的请求">七、Restful风格的请求</a><br/>
## 正文<br/>

## 一、搭建hello spring工程

### 1.1 项目搭建

1.新建 maven web 工程，并引入相应的依赖

```xml
 <properties>
        <spring-base-version>5.1.3.RELEASE</spring-base-version>
 </properties>

<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring-base-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring-base-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring-base-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring-base-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring-base-version}</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

2.得益于 servlet3.0 和 spring 的支持，我们可以在没有 web.xml 的情况下完成关于 servlet 配置。

   新建 DispatcherServletInitializer.java 文件,这个类的作用相当于我们在 xml 方式下 web.xml 中配置的 DispatcherServlet

```java
package com.heibaiying.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * @author : heibaiying
 */

public class DispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{ServletConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}

```

3.新建 ServletConfig.java，文件内容如下 (这个类相当于我们在 xml 配置方式中的 springApplication.xml)

```java
package com.heibaiying.config;

import com.heibaiying.exception.NoAuthExceptionResolver;
import com.heibaiying.interceptors.MyFirstInterceptor;
import com.heibaiying.interceptors.MySecondInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

import java.util.List;

/**
 * @author : heibaiying
 */
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {"com.heibaiying.controller"})
public class ServletConfig implements WebMvcConfigurer {

    /**
     * 配置视图解析器
     */
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
        internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
        internalResourceViewResolver.setSuffix(".jsp");
        internalResourceViewResolver.setExposeContextBeansAsAttributes(true);
        return internalResourceViewResolver;
    }

    /**
     * 配置静态资源处理器
     */
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

}

```

4.在 src 下新建 controller 用于测试

```java
package com.heibaiying.controller;

import com.heibaiying.exception.NoAuthException;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @author : heibaiying
 * @description : hello spring
 */

@Controller
@RequestMapping("mvc")
public class HelloController {

    @RequestMapping("hello")
    private String hello() {
        return "hello";
    }

}

```

5.在 WEB-INF 下新建 jsp 文件夹，新建 hello.jsp 文件

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    Hello Spring MVC!
</body>
</html>
```

6.启动 tomcat 服务，访问 localhost:8080/mvc/hello

### 1.2 相关注解说明

**1.@Configuration**

@Configuration 用于定义配置类，可替换 xml 配置文件，被注解的类内部包含有一个或多个被@Bean 注解的方法，这些方法将会被 AnnotationConfigApplicationContext 或 AnnotationConfigWebApplicationContext 类进行扫描，并用于构建 bean 定义，初始化 Spring 容器。

**2.@EnableWebMvc**

简单的说就是提供了部分 springmvc 的功能，例如格式转换和参数绑定。



## 二、配置自定义拦截器

1.创建自定义拦截器，实现接口 HandlerInterceptor（这里我们创建两个拦截器，用于测试拦截器方法的执行顺序）

```java
package com.heibaiying.interceptors;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author : heibaiying
 * @description : spring5 中 preHandle，postHandle，afterCompletion 在接口中被声明为默认方法
 */
public class MyFirstInterceptor implements HandlerInterceptor {

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        System.out.println("进入第一个拦截器 preHandle");
        return true;
    }

    // 需要注意的是，如果对应的程序报错，不一定会进入这个方法 但一定会进入 afterCompletion 这个方法
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        System.out.println("进入第一个拦截器 postHandle");
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("进入第一个拦截器 afterCompletion");
    }
}
```

```java
package com.heibaiying.interceptors;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author : heibaiying
 * @description : spring5 中 preHandle，postHandle，afterCompletion 在接口中被声明为默认方法
 */
public class MySecondInterceptor implements HandlerInterceptor {

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        System.out.println("进入第二个拦截器 preHandle");
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        System.out.println("进入第二个拦截器 postHandle");
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("进入第二个拦截器 afterCompletion");
    }
}

```

2.在 ServletConfig.java 中注册自定义拦截器

```java
 /**
  * 添加自定义拦截器
  */
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyFirstInterceptor()).addPathPatterns("/mvc/**").excludePathPatterns("mvc/login");
        registry.addInterceptor(new MySecondInterceptor()).addPathPatterns("/mvc/**");
    }
```

3.关于多个拦截器方法执行顺序的说明

拦截器的执行顺序是按声明的先后顺序执行的，先声明的拦截器中的 preHandle 方法会先执行，然而它的 postHandle 方法和 afterCompletion 方法却会后执行。



## 三、全局异常处理 

1.定义自定义异常

```java
package com.heibaiying.exception;

/**
 * @author : heibaiying
 * @description : 自定义无权限异常
 */
public class NoAuthException extends RuntimeException {

    public NoAuthException() {
        super();
    }

    public NoAuthException(String message) {
        super(message);
    }

    public NoAuthException(String message, Throwable cause) {
        super(message, cause);
    }

    public NoAuthException(Throwable cause) {
        super(cause);
    }

}

```

2.实现自定义异常处理器

```java
package com.heibaiying.exception;

import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author : heibaiying
 * @description : 无权限异常处理机制
 */
public class NoAuthExceptionResolver implements HandlerExceptionResolver {

    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        if (ex instanceof NoAuthException && !isAjax(request)) {
            return new ModelAndView("NoAuthPage");
        }
        return new ModelAndView();
    }

    // 判断是否是 Ajax 请求
    private boolean isAjax(HttpServletRequest request) {
        return "XMLHttpRequest".equalsIgnoreCase(request.getHeader("X-Requested-With"));
    }
}

```

3.在 ServletConfig.java 注册自定义异常处理器

```java
/**
* 添加全局异常处理器
*/
public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
    resolvers.add(new NoAuthExceptionResolver());
}
```

4.定义测试 controller，抛出自定义异常

```java
@Controller
@RequestMapping("mvc")
public class HelloController {

    @RequestMapping("hello")
    private String hello() {
        return "hello";
    }


    @RequestMapping("auth")
    private void auth() {
        throw new NoAuthException("没有对应的访问权限！");
    }
}
```

注：调用这个 controller 时，同时也可以验证在拦截器部分提到的：如果对应的程序报错，拦截器不一定会进入 postHandle 这个方法 但一定会进入 afterCompletion 这个方法



## 四、参数绑定

### 4.1 参数绑定

1.新建 Programmer.java

```java
package com.heibaiying.bean;

import lombok.Data;

/**
 * @author : heibaiying
 * @description :
 */
@Data
public class Programmer {

    private String name;

    private int age;

    private float salary;

    private String birthday;
}

```

注：@Data 是 lombok 包下的注解，用来生成相应的 set、get 方法，使得类的书写更为简洁。

2.新建 ParamBindController.java 文件

```java
package com.heibaiying.controller;

import com.heibaiying.bean.Programmer;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.format.datetime.DateFormatter;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.Date;

/**
 * @author : heibaiying
 * @description :参数绑定
 */
@Controller
public class ParamBindController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd HH:mm:ss"));
    }


    // 参数绑定与日期格式转换
    @RequestMapping("param")
    public String param(String name, int age, double salary, @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") Date birthday, Model model) {
        model.addAttribute("name", name);
        model.addAttribute("age", age);
        model.addAttribute("salary", salary);
        model.addAttribute("birthday", birthday);
        return "param";
    }

    @RequestMapping("param2")
    public String param2(String name, int age, double salary, Date birthday, Model model) {
        model.addAttribute("name", name);
        model.addAttribute("age", age);
        model.addAttribute("salary", salary);
        model.addAttribute("birthday", birthday);
        return "param";
    }


    @PostMapping("param3")
    public String param3(Programmer programmer, String extendParam, Model model) {
        System.out.println("extendParam" + extendParam);
        model.addAttribute("p", programmer);
        return "param";
    }

}

```

3.新建 param.jsp 文件

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Restful</title>
</head>
<body>
<ul>
    <li>姓名：${empty name ? p.name : name}</li>
    <li>年龄：${empty age ? p.age : age}</li>
    <li>薪酬：${empty salary ? p.salary : salary}</li>
    <li>生日：${empty birthday ? p.birthday : birthday}</li>
</ul>
</body>
</html>

```

4.启动 tomcat，用[postman](https://www.getpostman.com/) 软件发送请求进行测试

### 4.2 关于日期格式转换的三种方法

1.如上实例代码所示，在对应的 controller 中初始化绑定

```java
@InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd HH:mm:ss"));
    }
```

2.利用@DateTimeFormat 注解，如果是用实体类去接收参数，则在对应的属性上用@DateTimeFormat 和@JsonFormat 声明

```java
public String param(@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") Date birthday)
```

3.使用全局的日期格式绑定，新建自定义日期格式转化类，之后在 ServletConfig.java 中进行注册

```java
package com.heibaiying.convert;

import org.springframework.core.convert.converter.Converter;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @author : heibaiying
 * @description :
 */
public class CustomDateConverter implements Converter<String, Date> {

    public Date convert(String s) {
        try {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            return simpleDateFormat.parse(s);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}

```

ServletConfig.java

```java
/**
 * 添加全局日期处理
 */
public void addFormatters(FormatterRegistry registry) {
	registry.addConverter(new CustomDateConverter());
}
```



## 五、数据校验

1.spring 支持的数据校验是 JSR303 的标准，需要引入依赖的 jar 包

```java
 <!-- 数据校验依赖包 -->
      <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.0.13.Final</version>
        </dependency>
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>2.0.1.Final</version>
        </dependency>
```

2.新建测试 ParamValidController.java，主要是在需要校验的参数前加上@Validated，声明参数需要被校验，同时加上 bindingResult 参数，这个参数中包含了校验的结果

```java
package com.heibaiying.controller;

import com.heibaiying.bean.Programmer;
import org.hibernate.validator.constraints.Length;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.format.datetime.DateFormatter;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import java.util.Date;
import java.util.List;

/**
 * @author : heibaiying
 * @description :数据校验
 */
@RestController
public class ParamValidController {


    @PostMapping("validate")
    public void valid(@Validated Programmer programmer,
                      BindingResult bindingResult) {
        List<ObjectError> allErrors = bindingResult.getAllErrors();
        for (ObjectError error : allErrors) {
            System.out.println(error.getDefaultMessage());
        }
    }

}

```

3.在 Programmer.java 的对应属性上加上注解约束 (支持的注解可以在 javax.validation.constraints 包中查看)

```java
package com.heibaiying.bean;

import lombok.Data;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;

/**
 * @author : heibaiying
 * @description :
 */
@Data
public class Programmer {

    @NotNull
    private String name;

    @Min(value = 0,message = "年龄不能为负数！" )
    private int age;

    @Min(value = 0,message = "薪酬不能为负数！" )
    private float salary;

    private String birthday;
}

```

## 六、文件上传与下载

#### 6.1 文件上传

1.在 ServletConfig.java 中进行配置，使之支持文件上传

```java
/**
* 配置文件上传
*/
@Bean
public CommonsMultipartResolver multipartResolver(){
    CommonsMultipartResolver resolver = new CommonsMultipartResolver();
    resolver.setMaxUploadSize(1024*1000*10);
    resolver.setMaxUploadSizePerFile(1024*1000);
    resolver.setDefaultEncoding("utf-8");
    return resolver;
}
```

2.新建测试上传的 FileController.java

```java
package com.heibaiying.controller;

import com.heibaiying.utils.FileUtil;
import org.apache.commons.io.FileUtils;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpSession;
import java.io.File;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;

/**
 * @author : heibaiying
 * @description : 文件上传
 */

@Controller
public class FileController {

    @GetMapping("file")
    public String filePage() {
        return "file";
    }


    /***
     * 单文件上传
     */
    @PostMapping("upFile")
    public String upFile(MultipartFile file, HttpSession session) {
        //保存在项目根目录下 image 文件夹下，如果文件夹不存在则创建
        FileUtil.saveFile(file, session.getServletContext().getRealPath("/image"));
        // success.jsp 就是一个简单的成功页面
        return "success";
    }

    /***
     * 多文件上传 多个文件用同一个名字
     */
    @PostMapping("upFiles")
    public String upFiles(@RequestParam(name = "file") MultipartFile[] files, HttpSession session) {
        for (MultipartFile file : files) {
            FileUtil.saveFile(file, session.getServletContext().getRealPath("images"));
        }
        return "success";
    }

    /***
     * 多文件上传方式 2 分别为不同文件指定不同名字
     */
    @PostMapping("upFiles2")
    public String upFile(String extendParam,
                         @RequestParam(name = "file1") MultipartFile file1,
                         @RequestParam(name = "file2") MultipartFile file2, HttpSession session) {
        String realPath = session.getServletContext().getRealPath("images2");
        FileUtil.saveFile(file1, realPath);
        FileUtil.saveFile(file2, realPath);
        System.out.println("extendParam:" + extendParam);
        return "success";
    }
}

```

3.其中工具类 FileUtil.java 代码如下

```java
package com.heibaiying.utils;

import org.springframework.web.multipart.MultipartFile;
import java.io.*;

/**
 * @author : heibaiying
 * @description : 文件上传工具类
 */

public class FileUtil {

    public static String saveFile(MultipartFile file, String path) {
        InputStream inputStream = null;
        FileOutputStream outputStream = null;
        String fullPath = path + File.separator + file.getOriginalFilename();
        try {
            File saveDir = new File(path);
            if (!saveDir.exists()) {
                saveDir.mkdirs();
            }
            outputStream = new FileOutputStream(new File(fullPath));
            inputStream = file.getInputStream();
            byte[] bytes = new byte[1024 * 1024];
            int read;
            while ((read = inputStream.read(bytes)) != -1) {
                outputStream.write(bytes, 0, read);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (outputStream != null) {
                try {
                    outputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return fullPath;
    }

}
```

4.新建用于上传的 jsp 页面，上传文件时表单必须声明 enctype="multipart/form-data"

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>文件上传</title>
    <link rel="stylesheet" href="${pageContext.request.contextPath}/css/file.css">
</head>
<body>

    <form action="${pageContext.request.contextPath }/upFile" method="post" enctype="multipart/form-data">
        请选择上传文件：<input name="file" type="file"><br>
        <input type="submit" value="点击上传文件">
    </form>

    <form action="${pageContext.request.contextPath }/upFiles" method="post" enctype="multipart/form-data">
        请选择上传文件 (多选)：<input name="file" type="file" multiple><br>
        <input type="submit" value="点击上传文件">
    </form>

    <form action="${pageContext.request.contextPath }/upFiles2" method="post" enctype="multipart/form-data">
        请选择上传文件 1：<input name="file1" type="file"><br>
        请选择上传文件 2：<input name="file2" type="file"><br>
        文件内容额外备注: <input name="extendParam" type="text"><br>
        <input type="submit" value="点击上传文件">
    </form>

</body>
</html>

```

#### 6.2 文件下载

1.在 fileController.java 中加上方法：

```java
  /***
   * 上传用于下载的文件
   */
    @PostMapping("upFileForDownload")
    public String upFileForDownload(MultipartFile file, HttpSession session, Model model) throws UnsupportedEncodingException {
        String path = FileUtil.saveFile(file, session.getServletContext().getRealPath("/image"));
        model.addAttribute("filePath", URLEncoder.encode(path,"utf-8"));
        model.addAttribute("fileName", file.getOriginalFilename());
        return "fileDownload";
    }

    /***
     * 下载文件
     */
    @GetMapping("download")
    public ResponseEntity<byte[]> downloadFile(String filePath) throws IOException {
        HttpHeaders headers = new HttpHeaders();
        File file = new File(filePath);
        // 解决文件名中文乱码
        String fileName=new String(file.getName().getBytes("UTF-8"),"iso-8859-1");
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        headers.setContentDispositionFormData("attachment", fileName);

        return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file),
                headers, HttpStatus.CREATED);
    }
```

2.其中 fileDownload.jsp 如下：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>文件下载</title>
</head>
<body>
    <a href="${pageContext.request.contextPath}/download?filePath=${filePath}">${fileName}</a>
</body>
</html>
```



## 七、Restful风格的请求

1.新建 Pet.java 实体类

```java
package com.heibaiying.bean;

import lombok.Data;

/**
 * @author : heibaiying
 * @description :测试 restful 风格的实体类
 */

@Data
public class Pet {

    private String ownerId;

    private String petId;
}

```

2.新建 RestfulController.java，用@PathVariable 和@ModelAttribute 注解进行参数绑定。

注： 在 REST 中，资源通过 URL 进行识别和定位。REST 中的行为是通过 HTTP 方法定义的。在进行不同行为时对应 HTTP 方法和 Spring 注解分别如下：

- 创建资源时：POST（PostMapping）
- 读取资源时：GET（ @GetMapping）
- 更新资源时：PUT 或 PATCH（PutMapping、PatchMapping）
- 删除资源时：DELETE（DeleteMapping）

```java
package com.heibaiying.controller;

import com.heibaiying.bean.Pet;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

/**
 * @author : heibaiying
 * @description : Restful 风格的请求
 */

@RestController
public class RestfulController {

    @GetMapping("restful/owners/{ownerId}/pets/{petId}")
    public void get(@PathVariable String ownerId, @PathVariable String petId) {
        System.out.println("ownerId:" + ownerId);
        System.out.println("petId:" + petId);
    }

    @GetMapping("restful2/owners/{ownerId}/pets/{petId}")
    public void get(@ModelAttribute Pet pet) {
        System.out.println("ownerId:" + pet.getOwnerId());
        System.out.println("petId:" + pet.getPetId());
    }

}

```

