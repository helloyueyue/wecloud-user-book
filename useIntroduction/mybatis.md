# mybatis使用说明
## 引入mybatis相关jar包
```xml:pom.xml
<!-- mysql依赖 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!-- mybatis依赖 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.2</version>
</dependency>
## mybatis_generator自动生成mapper.xml的插件
<profiles>
    <profile>
        <id>mybatis_generator</id>
        <dependencies>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>
        </dependencies>
        <build>
            <finalName>mybatis_generator</finalName>
            <plugins>
                <plugin>
                    <groupId>org.mybatis.generator</groupId>
                    <artifactId>mybatis-generator-maven-plugin</artifactId>
                    <version>1.3.2</version>
                    <configuration>
                        <!--允许移动生成的文件-->
                        <verbose>true</verbose>
                        <!--允许覆盖生成的文件-->
                        <overwrite>true</overwrite>
                    </configuration>
                    <dependencies>
                        <dependency>
                            <groupId>mysql</groupId>
                            <artifactId>mysql-connector-java</artifactId>
                            <version>5.1.37</version>
                        </dependency>
                    </dependencies>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```
## 配置文件
```properties:application.properties
#mysql config
spring.datasource.url=jdbc:mysql://localhost:3306/wecloud_demo?useUnicode=true&amp;characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

#mybatis config
mybatis.mapperLocations=classpath:sqlmap/*Mapper.xml

#将mapper一级的日志级别设为debug模式可以打印sql语句
logging.level.com.navinfo.wecloud.demo.mapper=debug
```
```xml:generatorConfig.xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <plugin type="org.mybatis.generator.plugins.RowBoundsPlugin"></plugin>
        <!--去除注释  -->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--数据库连接-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/wecloud_demo?characterEncoding=utf8"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!--生成实体类 指定包名 以及生成的地址 （可以自定义地址，但是路径不存在不会自动创建  使用Maven生成在target目录下，会自动创建） -->
        <javaModelGenerator targetPackage="com.navinfo.wecloud.demo.bean"
                            targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!--生成SQLMAP文件 -->
        <sqlMapGenerator targetPackage="sqlmap" targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <!--生成Dao文件 可以配置 type="XMLMAPPER"生成xml的dao实现-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.navinfo.wecloud.demo.mapper"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        <!--对应数据库表 mysql可以加入主键自增 字段命名 忽略某字段等,开发者需要修改增加-->
        <table tableName="USERINFO" domainObjectName="UserInfo"></table>
    </context>
</generatorConfiguration>
```
## 类定义
定义UserInfo实体类和在wecloud_demo数据库中创建对应的userinfo表。<br/>
定义UserInfoMapper类：
```java:UserInfoMapper.java
package com.navinfo.wecloud.demo.mapper;

import com.navinfo.wecloud.demo.bean.UserInfo;
import com.navinfo.wecloud.demo.bean.UserInfoExample;
import java.util.List;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.session.RowBounds;

public interface UserInfoMapper {
    int countByExample(UserInfoExample example);

    int deleteByExample(UserInfoExample example);

    int deleteByPrimaryKey(Integer id);

    int insert(UserInfo record);

    int insertSelective(UserInfo record);

    List<UserInfo> selectByExampleWithRowbounds(UserInfoExample example, RowBounds rowBounds);

    List<UserInfo> selectByExample(UserInfoExample example);

    UserInfo selectByPrimaryKey(Integer id);

    int updateByExampleSelective(@Param("record") UserInfo record, @Param("example") UserInfoExample example);

    int updateByExample(@Param("record") UserInfo record, @Param("example") UserInfoExample example);

    int updateByPrimaryKeySelective(UserInfo record);

    int updateByPrimaryKey(UserInfo record);
}
```
Profiles中勾选mybatis_generator启动项目，会在resources的sqlmap文件夹下生成对应的UserInfoMapper.xml文件。<br/>
## Mybatis分页用法
定义分页插件：
```java:MySQLLimitPlugin.java
package com.navinfo.wecloud.demo.plugin;

import java.util.List;

import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.PluginAdapter;
import org.mybatis.generator.api.dom.java.Field;
import org.mybatis.generator.api.dom.java.FullyQualifiedJavaType;
import org.mybatis.generator.api.dom.java.JavaVisibility;
import org.mybatis.generator.api.dom.java.Method;
import org.mybatis.generator.api.dom.java.Parameter;
import org.mybatis.generator.api.dom.java.PrimitiveTypeWrapper;
import org.mybatis.generator.api.dom.java.TopLevelClass;
import org.mybatis.generator.api.dom.xml.Attribute;
import org.mybatis.generator.api.dom.xml.TextElement;
import org.mybatis.generator.api.dom.xml.XmlElement;

public class MySQLLimitPlugin extends PluginAdapter {

    @Override
    public boolean validate(List<String> list) {
        return true;
    }

    /**
     * 为每个Example类添加limit和offset属性已经set、get方法
     */
    @Override
    public boolean modelExampleClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {

        PrimitiveTypeWrapper integerWrapper = FullyQualifiedJavaType.getIntInstance().getPrimitiveTypeWrapper();

        Field limit = new Field();
        limit.setName("limit");
        limit.setVisibility(JavaVisibility.PRIVATE);
        limit.setType(integerWrapper);
        topLevelClass.addField(limit);

        Method setLimit = new Method();
        setLimit.setVisibility(JavaVisibility.PUBLIC);
        setLimit.setName("setLimit");
        setLimit.addParameter(new Parameter(integerWrapper, "limit"));
        setLimit.addBodyLine("this.limit = limit;");
        topLevelClass.addMethod(setLimit);

        Method getLimit = new Method();
        getLimit.setVisibility(JavaVisibility.PUBLIC);
        getLimit.setReturnType(integerWrapper);
        getLimit.setName("getLimit");
        getLimit.addBodyLine("return limit;");
        topLevelClass.addMethod(getLimit);

        Field offset = new Field();
        offset.setName("offset");
        offset.setVisibility(JavaVisibility.PRIVATE);
        offset.setType(integerWrapper);
        topLevelClass.addField(offset);

        Method setOffset = new Method();
        setOffset.setVisibility(JavaVisibility.PUBLIC);
        setOffset.setName("setOffset");
        setOffset.addParameter(new Parameter(integerWrapper, "offset"));
        setOffset.addBodyLine("this.offset = offset;");
        topLevelClass.addMethod(setOffset);

        Method getOffset = new Method();
        getOffset.setVisibility(JavaVisibility.PUBLIC);
        getOffset.setReturnType(integerWrapper);
        getOffset.setName("getOffset");
        getOffset.addBodyLine("return offset;");
        topLevelClass.addMethod(getOffset);

        return true;
    }

    /**
     * 为Mapper.xml的selectByExample添加limit
     */
    @Override
    public boolean sqlMapSelectByExampleWithoutBLOBsElementGenerated(XmlElement element,
                                                                     IntrospectedTable introspectedTable) {

        XmlElement ifLimitNotNullElement = new XmlElement("if");
        ifLimitNotNullElement.addAttribute(new Attribute("test", "limit != null"));

        XmlElement ifOffsetNotNullElement = new XmlElement("if");
        ifOffsetNotNullElement.addAttribute(new Attribute("test", "offset != null"));
        ifOffsetNotNullElement.addElement(new TextElement("limit ${offset}, ${limit}"));
        ifLimitNotNullElement.addElement(ifOffsetNotNullElement);

        XmlElement ifOffsetNullElement = new XmlElement("if");
        ifOffsetNullElement.addAttribute(new Attribute("test", "offset == null"));
        ifOffsetNullElement.addElement(new TextElement("limit ${limit}"));
        ifLimitNotNullElement.addElement(ifOffsetNullElement);

        element.addElement(ifLimitNotNullElement);

        return true;
    }
}
```
在UserInfoExample类中添加limit和offset属性以及对应set/get方法。
```java:UserInfoExample.java
private Integer limit;
private Integer offset;
public void setLimit(Integer limit) {
    this.limit = limit;
}
public Integer getLimit() {
    return limit;
}
public void setOffset(Integer offset) {
    this.offset = offset;
}
public Integer getOffset() {
    return offset;
}
```
分页查询代码示例：
```
public List<UserInfo> queryList(String keyword, Integer page, Integer pageSize) {
    UserInfoExample example = new UserInfoExample();
    example.setLimit(pageSize); // page size limit
    example.setOffset(pageSize*(page-1)); // offset
    UserInfoExample.Criteria criteria = example.createCriteria();
    criteria.andUsernameLike(keyword + "%");
    List<UserInfo> list = userInfoMapper.selectByExample(example);
    return list;
}
```






