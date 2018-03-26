+++

author = "飞狐"
categories = ["Java编程","Spring系列"]
tags = ["Java","Spring","Spring Security"]
date = "2016-03-20T16:05:52+08:00"
description = "本文描述了如何利用Spring Security来在我们的程序中动态的改变用户的权限"
keywords = ["Java","Spring","Spring Security"]
title = "利用Spring Security动态的改变权限"

+++

利用 **[Spring Security](http://projects.spring.io/spring-security/)** 来管理我们的web程序时，通常需要在***UserDetailsService*** 接口中的 ***loadUserByUsername*** 方法中来初始化权限信息,但 ***UserDetailsService*** 一般用于登录验证，这也意味着用户的权限在登录过程中就会被计算出来。通常情况下由于用户的权限很少发生变化，在登录过程中计算出用户权限是合理的，但有些情况下，我们需要在中途来动态的改变用户的权限，此时我们可以利用 **[Spring Security](http://projects.spring.io/spring-security/)** 提供的API来实现。
<!--more-->

以我自己的项目为例，***UserDetailsService*** 接口中的 ***loadUserByUsername*** 具体实现如下：
```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
       
	UserModel userModel=userDao.getUserByUsername(username);
       
	if(userModel==null){
		throw new UsernameNotFoundException(username+" not exist!");
	}
	
	List<GrantedAuthority> userAuthList=new ArrayList<GrantedAuthority>();
    
	//查询出用户相关的所有权限并放入List中
	List<AuthorityVO> authList=authorityDao.queryAuthorityByUserId(userModel.getId());
	for(AuthorityVO authVO:authList){
		userAuthList.add(new SimpleGrantedAuthority(authVO.getAuthName()));
	}

	//将查询出来的权限赋予用户
	UserDetails userDetails=new User(userModel.getUsername(),userModel.getPassword(),true,true,true,true,userAuthList);
	
	return userDetails;
}
```
上述代码会一次性的把用户权限查询出来然后放入特定的 **session** 中，但是 ***UserDetailService*** 方法一般只在用户登录web系统成功时才会被调用一次，使用范围较为局限，有时候我们需要在用户使用的过程中动态的改变用户的权限（譬如在我自己的项目中，当用户选中不同的项目之后，不同的项目对应不同的权限）。利用 **[Spring Security](http://projects.spring.io/spring-security/)** 来管理权限信息时，用户的权限本质上是存储在一个 **session** 中，只不过被**[Spring Security](http://projects.spring.io/spring-security/)**进行了进一步的封装而已。所以若想动态的改变用户的权限，我们只需要将用户的信息重新存储到 **session** 中即可，具体代码如下所示：

```java
List<GrantedAuthority> authList=new ArrayList<GrantedAuthority>();//用于存储修改之后的权限列表
authList.add(new SimpleGrantedAuthority("addUser"));
authList.add(new SimpleGrantedAuthority("editUser"));

SecurityContext context=SecurityContextHolder.getContext();

UserDetails userDetails=(UserDetails) context.getAuthentication().getPrincipal();
Authentication auth=new UsernamePasswordAuthenticationToken(userDetails,userDetails.getPassword(),authList);

context.setAuthentication(auth); //重新设置上下文中存储的用户权限
```