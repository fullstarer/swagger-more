# Swagger-more


## 关于项目

​		公司内测试需要dubbo接口提供文档， 实习的闲暇之余有好奇Swagger的代码，看看它怎么大致的流程如何。之前又嫌单测跑用例麻烦，后面写了一个调用Spring Bean方法的Controller 。因此催生了Swagger-more。**重在扩展，代码不多**



## 支持的功能

1. 自动生成dubbo接口文档，同springfox 的JSON API
2. 接口支持调试，支持泛型调用，支持多对象传参
3. 支持javadoc生成接口信息的注解
4. 符合java接口文档的新UI




## 如何接入


### 一、引入依赖

1. 在Api 或者 实体类模块中引入注解的依赖, 整个包只有一个@ApiMethod的注解（将接口和实现分离，保证Api模块的独立）

```xml
<dependency>
    <groupId>com.github.uhfun</groupId>
    <artifactId>swagger-more-annotations</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

2. 在Server服务层里引入核心的依赖

```xml
<dependency>
     <groupId>com.github.uhfun</groupId>
    <artifactId>swagger-more-core</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

3. 如果需要使用接口方法的Javadoc 生成文档在Server服务层里引入支持javadoc的依赖
  在api包的pom里加上 plugin

  * 1) 替换 "your.app.api.package" 为你自己的包名

    ​	例如 com.mygroup.my-porject-api

    ​	如果有多个用 「,」或「:」或「;」 隔开

  * 2) 替换项目API模块 的 your.groupId、your.artifacId、your.version

   ````xml
   <docletArtifact>
   	<groupId>your.groupId</groupId>
   	<artifactId>your.artifactId</artifactId>   
   	<version>your.version</version>
   </docletArtifact>
   ````



**具体配置如下**

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-javadoc-plugin</artifactId>
      <version>3.1.1</version>
      <executions>
        <execution>
          <goals>
            <goal>javadoc</goal>
          </goals>
          <phase>process-classes</phase>
          <configuration>
            <doclet>com.github.uhfun.swagger.doclet.SwaggerMoreDoclet</doclet>
            <docletArtifacts>
              <docletArtifact>
                 <groupId>com.github.uhfun</groupId>
                 <artifactId>swagger-more-javadoc</artifactId>
                 <version>1.0.0-SNAPSHOT</version>
              </docletArtifact>
              <docletArtifact>
                <groupId>your.groupId</groupId>
                <artifactId>your.artifactId</artifactId>
                <version>your.version</version>
              </docletArtifact>
            </docletArtifacts>
            <additionalOptions>-classDir ${project.build.outputDirectory}</additionalOptions>
            <sourcepath>${project.build.sourceDirectory}</sourcepath>
            <subpackages>your.app.api.package</subpackages>
            <useStandardDocletOptions>false</useStandardDocletOptions>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```



### 二、添加配置

```java
@Configuration
@EnableSwaggerMore
public class SwaggerConfig {
}
```



### 三、添加注解

#### 方法注解

结构如下

为了文档清晰 设置name， name为请求时的参数   value为解释

[@ApiParam ](https://springfox.github.io/springfox/docs/current/#overriding-resolver-via-properties) 中的其他可以注释参考官方文档  一般用不到

```java
@Api(tags = "用户API")
public interface UserService {

    @ApiMethod(value = "保存用户", params = {
            @ApiParam(name = "user", value = "用户")
    })
    String save(User user);
}
```

> 如果引入了 swagger-more-javadoc , 可以将注解替换为注释


```java
/**
 * 用户API
 *
 * @author uhfun
 */
public interface UserService {

    /**
     * 保存用户
     *
     * @param user         用户
     * @return id
     */
    String save(User user);
}
```



#### 实体类注解

value为解释 name可以不用设置， Swagger在解析的时候可以获取到

[@ApiModelProperty ](https://springfox.github.io/springfox/docs/current/#overriding-resolver-via-properties) 中的参数可以参考官方文档或者里面的官方注释

```java
@ApiModel(description = "用户")
public class User implements Serializable {

    private static final long serialVersionUID = -7182552932351577562L;
    /**
     * 用户id
     */
    @ApiModelProperty(name = "id", required = true, example = "123456789")
    private String id;
    /**
     * 用户名称
     */
    @ApiModelProperty(value = "用户名称", required = true, example = "uhfun")
    private String name;
}
```



## FAQ


### 一、配置

需要开启或者在配置文件中配置**SpringMVC** 能够进行**http**请求的访问

```java
@EnableWebMvc
@Configuration
```


### 二、Dubbo接口注册的时候，bean 名称为类名（第一个字母小写）

因为Swagger-more中自定义Controller 接收请求的时候直接按类名获取bean，一般也都是这个命名，所以暂时没做处理...


## 版权信息

```
Copyright (c) 2019 uhfun

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```