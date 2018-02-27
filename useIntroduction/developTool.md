# 开发环境
推荐开发工具：Intelli IDEA

## 在Intelli IDEA下创建maven项目
![](/assets/img1.png)
![](/assets/img2.png)
![](/assets/img3.png)

## pom.xml引入Wecloud底层Jar包
```
<parent>
    <groupId>com.navinfo.wecloud</groupId>
    <artifactId>wecloud-parent</artifactId>
    <version>1.14-beta</version>
</parent>
<dependencies>
    <dependency>
        <groupId>com.navinfo.wecloud</groupId>
        <artifactId>wecloud-starter</artifactId>
    </dependency>
</dependencies>
```
