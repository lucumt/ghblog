+++

author = "飞狐"
categories = ["Java编程"]
tags = ["Java","MyBatis"]
date = "2016-05-30T18:20:37+08:00"
description = "Blog of Rosen Lu"
keywords = ["Java","MyBatis"]
title = "mybatis batch insert exception的解决方法"

+++

在利用 **[MyBatis](http://www.mybatis.org/mybatis-3/)** 进行多条数据插入时，为了提高性能我们可能会使用批量插入的功能来实现。示例代码如下:

* SQL配置文件:

	```xml
	<insert id="addAuthorityRoleBatch" parameterType="List">
	    INSERT INTO system_authority_role(role_id,authority_id)
	      VALUES
	      <foreach collection="list" item="authRole" separator=",">
	        (#{authRole.roleId},#{authRole.authorityId})
	      </foreach>
	  </insert>
	```

[//]:(设置前面的内容为summary)
<!--more-->

* Java代码:

	```java
	   public void adjustRoleAuth(String roleId, String authIdsStr) {
			authRoleDao.deleteAuthorityRoleByRole(roleId);
			String[] authIds=authIdsStr.split(";");
			List<AuthorityRoleModel> authRoleList=new ArrayList<AuthorityRoleModel>();
			for(String authId:authIds){
				authRoleList.add(new AuthorityRoleModel(roleId,authId));
			}
			authRoleDao.addAuthorityRoleBatch(authRoleList);
	  }
	```

上面的代码大多数时候可以正常运行，但是偶尔会出现如下异常：

```java
### SQL: INSERT INTO system_authority_role(role_id,authority_id)       VALUES
### Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 2
; bad SQL grammar []; nested exception is com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 2
at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.doTranslate(SQLErrorCodeSQLExceptionTranslator.java:233
```

上面的异常堆栈信息显示现在执行的MySQL语句发生了语法错误，INSERT VALUE后面的值为空，由于该问题有时候发生，有时候不发生，给我们分析该问题造成了一定的困扰。**该问题产生的根源为批量插入时的集合数据为空，使得SQL配置文件中的foreach循环没有执行，从而导致SQL语句不完整，进而产生该异常。**为了解决该问题我们可以批量插入之前先检查List数据集合是否为空，只有在不为空的情况下才进行插入，如下所示：

```java
public void adjustRoleAuth(String roleId, String authIdsStr) {
	authRoleDao.deleteAuthorityRoleByRole(roleId);
	String[] authIds=authIdsStr.split(";");
	List<AuthorityRoleModel> authRoleList=new ArrayList<AuthorityRoleModel>();
	for(String authId:authIds){
		authRoleList.add(new AuthorityRoleModel(roleId,authId));
	}
	if(authRoleList.size()>0){//只有在List不为空时才进行插入
		authRoleDao.addAuthorityRoleBatch(authRoleList);		
	}
}
```