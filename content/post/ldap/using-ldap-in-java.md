---
title: "在Java代码中集成LDAP"
date: 2021-12-26T09:15:58+08:00
lastmod: 2021-12-26T09:15:58+08:00
draft: false
keywords: ["java","ldap","ldaps","认证"]
description: "基于个人使用经验，简要记录如何在Java代码中集成LDAP/LDAPS,供后续参考使用"
tags: ["ldap"]
categories: ["java编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: true
codeTabSeperator: "::"

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

简单记录个人项目中基于`Java`使用[**LDAP**](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)的相关经验，供后续参考。

<!--more-->

## Java实现

`JDK`自带类库中的[**javax.naming**](https://docs.oracle.com/javase/8/docs/api/javax/naming/package-summary.html)包默认提供了`LDAP`的原生支持，其使用方式类似如下

{{% codetabs %}}  

```java::用户认证
public boolean ldapAuth(String username, String password) throws NamingException {
    boolean loginResult = false;
    
    Hashtable<String, String> adminEnv = new Hashtable<>();
    adminEnv.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
    adminEnv.put(Context.PROVIDER_URL, "ldap://ldap.lucumt.com");
    adminEnv.put(Context.SECURITY_AUTHENTICATION, "simple");
    adminEnv.put(Context.SECURITY_PRINCIPAL, "cn=admin,ou=adminUsers,ou=sso,dc=lucumt,dc=com");
    adminEnv.put(Context.SECURITY_CREDENTIALS, "qWertFx@");

    DirContext adminCtx = new InitialDirContext(adminEnv);

    String searchBase = "ou=sso,dc=lucumt,dc=com";
    String searchFilter = "cn=" + username + "";
    SearchControls controls = new SearchControls();
    controls.setSearchScope(SearchControls.SUBTREE_SCOPE);

    NamingEnumeration<SearchResult> results =
            adminCtx.search(searchBase, searchFilter, controls);

    if (results.hasMore()) {
        SearchResult result = results.next();
        String foundUserDn = result.getNameInNamespace();

        // 3. 使用搜索到的用户 DN 进行认证
        Hashtable<String, String> userEnv = new Hashtable<>();
        userEnv.putAll(adminEnv);
        userEnv.put(Context.SECURITY_PRINCIPAL, foundUserDn);
        userEnv.put(Context.SECURITY_CREDENTIALS, password);

        DirContext userCtx = new InitialDirContext(userEnv);
        System.out.println("认证成功！用户 DN：" + foundUserDn);
        userCtx.close();
        loginResult = true;
    } else {
        System.err.println("用户不存在！");
    }

    adminCtx.close();
    return loginResult;
}
```

```java::用户查询
public static void testLDAP() {
    // LDAP 服务器地址
    String ldapUrl = "ldap://ldap.lucumt.com";
    // LDAP 管理员的用户名
    String ldapUser = "cn=admin,ou=adminUsers,ou=sso,dc=lucumt,dc=com";
    // LDAP 管理员的密码
    String ldapPassword = "qWertFx@";
    // 搜索的基础 DN
    String baseDN = "ou=sso,dc=lucumt,dc=com";
    // 搜索的过滤器
    String searchFilter = "(objectClass=person)";

    // 配置 LDAP 环境
    Hashtable < String, String > env = new Hashtable < > ();
    env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
    env.put(Context.PROVIDER_URL, ldapUrl);
    env.put(Context.SECURITY_AUTHENTICATION, "simple");
    env.put(Context.SECURITY_PRINCIPAL, ldapUser);
    env.put(Context.SECURITY_CREDENTIALS, ldapPassword);

    try {
        // 创建 LDAP 上下文
        DirContext ctx = new InitialDirContext(env);
        // 设置搜索控件
        SearchControls searchControls = new SearchControls();
        searchControls.setSearchScope(SearchControls.SUBTREE_SCOPE);
        searchControls.setReturningAttributes(new String[] {
            "cn",
            "sn",
            "mail"
        }); // 要返回的属性

        // 执行搜索
        NamingEnumeration < SearchResult > results = ctx.search(baseDN, searchFilter, searchControls);

        while (results.hasMore()) {
            SearchResult result = results.next();
            Attributes attributes = result.getAttributes();

            Attribute cn = attributes.get("cn");
            Attribute sn = attributes.get("sn");
            Attribute mail = attributes.get("mail");

            System.out.println("CN: " + (cn != null ? cn.get() : "N/A"));
            System.out.println("SN: " + (sn != null ? sn.get() : "N/A"));
            System.out.println("Mail: " + (mail != null ? mail.get() : "N/A"));
            System.out.println("----------------------------");
        }

        // 关闭上下文
        ctx.close();

    } catch (NamingException e) {
        e.printStackTrace();
    }
}
```

{{% /codetabs %}}  

## Spring实现

`JDK`自带的`LDAP` 操作起来较为复杂，可利用`Spring`提供的[**LdapTemplate**](https://docs.spring.io/spring-ldap/docs/current/api/org/springframework/ldap/core/LdapTemplate.html)简化上述操作。

仿照下述代码引入相关依赖与配置

{{% codetabs %}}  

```xml::LDAP依赖引入
<dependency>
    <groupId>org.springframework.ldap</groupId>
    <artifactId>spring-ldap-core</artifactId>
</dependency>
```

```yaml::LDAP服务配置
spring:
  ldap:
    urls: ldap://ldap.lucumt.com
    base: ou=sso,dc=lucumt,dc=com
    username: cn=admin,ou=adminUsers,ou=sso,dc=lucumt,dc=com
    password: qWertFx@
```

```yaml::LDAP属性配置
ldap:
   mail: email
   userName: cn
   realName: displayName
   phone: mobile
```

{{% /codetabs %}}  

之后的代码如下所示，可看出同原始的`Java` 实现相比较，其篇幅已经缩减不少(高亮部分为核心代码)

{{% codetabs %}}  

```java::业务类 {data-line="13-14,22,32"}
import com.lucumt.app.config.LdapConfig;
import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.codec.digest.DigestUtils;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class AuthServiceImpl implements IAuthService {

    @Resource
    private LdapTemplate ldapTemplate;

    @Resource
    private LdapConfig ldapConfig;

    @Override
    public ResponseVM ldapLogin(UserVO loginUser) {
        String filter = ldapConfig.getUid() + "=" + loginUser.getUserName();
        boolean success = ldapTemplate.authenticate("", filter, loginUser.getPassword());
        if (!success) {
            return ResponseVM.fail("用户名或密码错误");
        }
        return ResponseVM.success("用户登录成功");
    }
    
    @Override
    public User fetchUserInfo(String username) {
        String filter = ldapConfig.getUid() + "=" + username;
        return ldapTemplate.searchForObject("", filter, context -> {
            DirContextAdapter ctx = (DirContextAdapter) context;
            User u = new User();
            u.setUserName(ctx.getStringAttribute(ldapConfig.getUsername()));
            u.setRealName(this.convertAttributeValue(ctx.getStringAttribute(ldapConfig.getRealName())));
            u.setEmail(this.convertAttributeValue(ctx.getStringAttribute(ldapConfig.getMail())));
            u.setPhone(this.convertAttributeValue(ctx.getStringAttribute(ldapConfig.getPhone())));
            return u;
        });
    }

}
```

```java::配置类
import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Getter
@Setter
@Component
@ConfigurationProperties(prefix = "ldap")
public class LdapConfig {

    private String uid;
    private String userName;
    private String realName;
    private String mail;
    private String phone;
}
```

{{% /codetabs %}}  

## 动态实现

对于某些标准化产品而言，需要交付到客户使用环境后进行测试与配置，此时需要提供`LDAP`动态配置的功能

![LDAP动态配置](/blog_img/ldap/using-ldap-in-java/ldap-dynamic-config.png "LDAP动态配置") 

可利用类似如下代码基于不同的配置动态的创建`LdapTemplate`，核心为通过`new`关键字重新创建相关对象

```java{data-line="2,15"}
private LdapTemplate createLdapTemplate(LdapConfigVO config) {
    LdapContextSource source = new LdapContextSource();
    source.setUrl(config.getUrls());
    source.setBase(config.getBase());

    //非匿名验证时需要设置管理员账号信息
    if (!config.isAnonymous()) {
        source.setUserDn(config.getUsername());
        source.setPassword(config.getPassword());
    }

    //确保上述配置生效
    source.afterPropertiesSet();

    LdapTemplate template = new LdapTemplate(source);
    return template;
}
```

## LDAPS支持

部分场景需要使用基于`SSL/TSL`实现的`LDAPS`，要支持此功能只需做如下2处修改即可

1.将`LDAP`的地址修改为加密形式，类似`ldaps://ldap.lucumt.com`

2.利用下述代码导入`LDAPS`的证书

```bash
# Windows导入与验证
keytool -import -alias ldap-ca -file ldap-lucumt.crt -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -storepass Pass@Word
keytool -list -keystore "%JAVA_HOME%\lib\security\cacerts" -storepass Pass@Word -alias ldap-ca

# Linux导入与验证
keytool -import -alias ldap-ca -file ldap-lucumt.crt -keystore "${JAVA_HOME}/jre/lib/security/cacerts" -storepass Pass@Word
keytool -list -keystore "${JAVA_HOME}/lib/security/cacerts" -storepass Pass@Word -alias ldap-ca
```

导入过程中的正常输出类似如下

![LDAP证书导入结果](/blog_img/ldap/using-ldap-in-java/ldap-certificate-import-result.png "LDAP证书导入结果") 

证书查看结果类似如下

![LDAP证书查看结果](/blog_img/ldap/using-ldap-in-java/ldap-certificate-list-result.png "LDAP证书查看结果") 