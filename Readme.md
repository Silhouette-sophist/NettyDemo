# Maven工程结构
## pom.xml文件分析
pom.xml文件包含了工程的各种信息， 通过分析可以知道Maven项目的关键概念：
- 父工程信息：SpringBoot之所以无需配置即可启动，实际上是在父工程中有相应默认配置。工程的解析和启动都依赖于父工程。
- 当前工程gav信息：当前工程特征信息。
- 通用属性设置：jdk版本属性等
- 依赖信息：依赖的三方jar信息
- 插件信息：编译时调用的maven插件内容，会在编译期对代码进行处理，如lombok会在编译期基于各种注解生成各类代码。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!--父项目中配置了很多文件，所以SpringBoot无需配置多少东西就可以执行了-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <!--当前项目的gav信息-->
    <groupId>com.example</groupId>
    <artifactId>NettyDemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>NettyDemo</name>
    <description>NettyDemo</description>

    <!--通用属性，java版本选择1.8-->
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--依赖信息-->
    <dependencies>
        <!--springboot-web starter-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!--springboot-test starter, 测试方法可在正常SpringBoot启动环境下执行case-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!--编译期会被执行的插件，比如Lombok会基于各种注解生成相应的java代码！-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

### 默认配置问题
直接父工程：spring-boot-starter-parent
maven仓库查看依赖信息：https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-parent/2.7.4
**注意**：实际上就是一个pom文件
![img.png](img.png)
**作用**：Parent pom providing dependency and plugin management for applications built with Maven。主要是配置插件和编译相关的内容

再看看祖父工程：spring-boot-dependencies
https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-dependencies/2.7.4
**注意**：实际上就是一个pom文件
![img_1.png](img_1.png)
**作用**：Spring Boot Dependencies。主要是配置SpringBoot使用的依赖。

**注意**：SpringBoot一个具体版本实际上就是一系列配置和依赖（包括各个依赖的默认版本信息！）的组合，用户使用即指定父项目即会融合pom文件，然后得到最终的pom文件。

>> 父项目解决了第一个问题：为何SpringBoot无需配置即可使用？因为会融合父项目的pom文件，完成编译插件和依赖配置等内容的默认行为。

```text
maven带来了项目继承的管理
 Maven Project可以理解为父工程。
Maven Module可以理解为子工程。
创建Maven Module工程必须有存在的父工程，maven就是通过父子工程进行工程管理的。
我们以项目为例子。onlinestore是一个主项目，onlinestore-core、onlinestore-interf、 onlinestore-chinaweb、onlinestore-americaweb是onlinestore的4个子项目（其中前两个是java 项目，后边两个是web项目）。
父项目和子项目在MyEclipse里边是平级的关系，在磁盘的目录结构中，子项目在父项目所在的文件夹中。

在pom.xml文件中，我们打开父项目的pom.xml文件，里边可以找到：
    <modules>
    <module>onlinestore-core</module>
    <module>onlinestore-interf</module>
    <module>onlinestore-chinaweb</module>
    <module>onlinestore-americaweb</module>
    </modules>

另外，在4个子项目的pom.xml的文件中，也能找到类似的：
    <parent>
    <artifactId>parent</artifactId>
    <groupId>com.uuwit.onlinestore</groupId>
    <version>0.0.1-SNAPSHOT</version>
    </parent>

也就是说，在父项目和子项目中，都有说明他们之间的关系。
通常一个大的项目会将项目分成多个模块，比如上边咱们举的例 子，core是项目的一些核心类或者组件，interf放接口类，其余两个分别是两个相对独立的web模块。正常情况下，两个web模块比如会依赖 core和interf这两个项目。而两个web项目之间通常并不会有依赖关系，但是他们之间却有很多共性的东西，比如说很多类似的配置、很多类似的 jar包等等，这时候父项目和子项目的优势就能体现出来了。我们知道maven父项目和子项目的pom.xml是有继承关系的，也就是说各个模块相同的部 分，我们可以配置到父项目的pom.xml文件中，这样子项目中的pom.xml只放自己个性的东西就可以了，这大大减少了工作量。另外，在编译和打包等 其他阶段，都可以统一在父项目中来进行，maven会自动操作其中的子项目，提高了效率。
【注意：】事实上，所有的maven项目都会继承一个超级pom，这个pom就是%M2_HOME%\lib\maven-2.2.1-uber.jar中的org\apache\maven\project\pom-4.0.0.xml。
所以 如果我们要建立 父子关系 
只要在 主项目的pom.xml中添加相应的 modules 说明 
以及在子项目中添加 parent说明即可。
```

### 代码执行流程

- 启动入口
```java
@SpringBootApplication
public class NettyDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(NettyDemoApplication.class, args);
    }
}
```

跟进spring-boot:2.7.4包中的SpringApplication.java
```java
public class SpringApplication {
    public SpringApplication(Class<?>... primarySources) {
        this((ResourceLoader)null, primarySources);
    }
    
    public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }

    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
    
    public ConfigurableApplicationContext run(String... args) {
        long startTime = System.nanoTime();
        DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
        ConfigurableApplicationContext context = null;
        this.configureHeadlessProperty();
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting(bootstrapContext, this.mainApplicationClass);

        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            
            Banner printedBanner = this.printBanner(environment);
            // 创建的ApplicationContext，即IOC容器？
            context = this.createApplicationContext();
            context.setApplicationStartup(this.applicationStartup);
            this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
            }

            listeners.started(context, timeTakenToStartup);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var12) {
            this.handleRunFailure(context, var12, listeners);
            throw new IllegalStateException(var12);
        }

        try {
            Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
            listeners.ready(context, timeTakenToReady);
            return context;
        } catch (Throwable var11) {
            this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var11);
        }
    }

    // 创建ApplicationContext
    protected ConfigurableApplicationContext createApplicationContext() {
        return this.applicationContextFactory.create(this.webApplicationType);
    }
}
```

ApplicationContextFactory中
```java
@FunctionalInterface
public interface ApplicationContextFactory {
    ApplicationContextFactory DEFAULT = (webApplicationType) -> {
        try {
            Iterator var1 = SpringFactoriesLoader.loadFactories(ApplicationContextFactory.class, ApplicationContextFactory.class.getClassLoader()).iterator();

            ConfigurableApplicationContext context;
            do {
                if (!var1.hasNext()) {
                    return new AnnotationConfigApplicationContext();
                }

                ApplicationContextFactory candidate = (ApplicationContextFactory)var1.next();
                context = candidate.create(webApplicationType);
            } while(context == null);

            return context;
        } catch (Exception var4) {
            throw new IllegalStateException("Unable create a default ApplicationContext instance, you may need a custom ApplicationContextFactory", var4);
        }
    };

    ConfigurableApplicationContext create(WebApplicationType webApplicationType);

    static ApplicationContextFactory ofContextClass(Class<? extends ConfigurableApplicationContext> contextClass) {
        return of(() -> {
            return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
        });
    }

    static ApplicationContextFactory of(Supplier<ConfigurableApplicationContext> supplier) {
        return (webApplicationType) -> {
            return (ConfigurableApplicationContext)supplier.get();
        };
    }
}
```




