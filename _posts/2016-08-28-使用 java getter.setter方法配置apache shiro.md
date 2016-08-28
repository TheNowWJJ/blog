---
layout: post_layout
title: 基于java getter/setter方法配置apache shiro
time: 2016年08月28日 星期六
location: 济南
pulished: true
excerpt_separator: "```"
---

* 说明
	* 谷歌,百度上介绍apache shiro配置的文章并不少,但是缺少我想要的基于java getter/setter方法配置apache shiro的文章.今天整一个.
	* 重点在于 ShiroConfig类中如何配置shiroFilter,securityManager,tokenRealm,sessionListener以及sessionManager.
	* *@Configuration*是spring的一个注解,用来标识该类是一个配置类,并且会注入到spring容器中.可以直接使用@Autowird注入.
* 代码
	SessionListener:
	```
	import org.apache.shiro.session.Session;
	import org.springframework.stereotype.Component;
	import ss.app.common.util.ContextUtil;

	@Component
	public class SessionListener implements org.apache.shiro.session.SessionListener {

	  @Override
	  public void onStart(Session session) {
	  }

	  @Override
	  public void onStop(Session session) {
	  }

	  @Override
	  public void onExpiration(Session session) {
	  }
	}
	```

	UserRealm:
	```
	import org.apache.commons.lang3.StringUtils;
	import org.apache.shiro.authc.*;
	import org.apache.shiro.authz.AuthorizationInfo;
	import org.apache.shiro.realm.AuthorizingRealm;
	import org.apache.shiro.subject.PrincipalCollection;
	import org.springframework.beans.factory.annotation.Autowired;
	import ss.app.user.entity.SystemUser;
	import ss.app.user.service.SystemUserService;

	public class UserRealm extends AuthorizingRealm {

	  @Autowired
	  private SystemUserService systemUserService;

	  @Override
	  protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
	    return null;
	  }

	  @Override
	  protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
	    String name = (String) token.getPrincipal();
	    String password = String.valueOf((char[]) token.getCredentials());

	    if (StringUtils.isBlank("name") || StringUtils.isBlank("password")) {
	      throw new UnknownAccountException("用户名或者密码为空");
	    }

	    SystemUser user = systemUserService.findByName(name);

	    if (null == user) {
	      throw new UnknownAccountException("用户不存在");
	    }

	    if (!StringUtils.equals(password, user.getPassword())) {
	      throw new UnknownAccountException("密码不正确");
	    }

	    //如果身份认证验证成功，返回一个AuthenticationInfo实现
	    return new SimpleAuthenticationInfo(name, password, getName());
	  }
	}
	```
	ShiroConfig:
	```
	import org.apache.shiro.realm.Realm;
	import org.apache.shiro.session.SessionListener;
	import org.apache.shiro.session.mgt.SessionManager;
	import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
	import org.apache.shiro.web.filter.authc.UserFilter;
	import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
	import org.apache.shiro.web.session.mgt.DefaultWebSessionManager;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;

	import javax.servlet.Filter;
	import java.util.*;

	@Configuration
	public class ShiroConfig {

	  @Bean(name = "shiroFilter")
	  public ShiroFilterFactoryBean shiroFilter() {
	    ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
	    shiroFilter.setLoginUrl("/");
	    shiroFilter.setSuccessUrl("/index");
	    shiroFilter.setUnauthorizedUrl("/forbidden");

	    Map<String, Filter> filters = new HashMap<>();
	    filters.put("users", new UserFilter());
	    shiroFilter.setFilters(filters);

	    Map<String, String> filterChainMap = new HashMap<>();
	    filterChainMap.put("/index/**", "users");
	    filterChainMap.put("/menu/**", "users");
	    filterChainMap.put("/gzhyy/**", "users");
	    filterChainMap.put("/gzhgl/**", "users");
	    filterChainMap.put("/assert/**", "anon");
	    filterChainMap.put("/css/**", "anon");
	    filterChainMap.put("/js/**", "anon");
	    shiroFilter.setFilterChainDefinitionMap(filterChainMap);
	    shiroFilter.setSecurityManager(securityManager());

	    return shiroFilter;
	  }

	  public org.apache.shiro.mgt.SecurityManager securityManager() {
	    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
	    securityManager.setSessionManager(sessionManager());
	    Collection<Realm> realms = new ArrayList<>();
	    realms.add(tokenRealm());
	    securityManager.setRealms(realms);
	    return securityManager;
	  }

	  @Bean
	  public UserRealm tokenRealm() {
	    return new UserRealm();
	  }

	  public List<SessionListener> sessionListener() {
	    List<SessionListener> listeners = new ArrayList<>();
	    listeners.add(new ss.app.common.config.session.SessionListener());
	    return listeners;
	  }

	  public SessionManager sessionManager() {
	    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
	    sessionManager.setSessionListeners(sessionListener());
	    sessionManager.setGlobalSessionTimeout(30 * 60　* 1000); // 30分钟
	    return sessionManager;
	  }
	}

	```


