#http auth basic  最基本的http验证
C:\Users\Administrator>curl -H "Authorization:Basic dXNlcjoxMjM=" -i http://localhost:8080
HTTP/1.1 200
Set-Cookie: JSESSIONID=37C838C7BE7BBD40CB4A13EF0B0DCDE0; Path=/; HttpOnly
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: text/plain;charset=UTF-8
Content-Length: 22
Date: Tue, 08 Dec 2020 05:55:18 GMT

hello, spring security


C:\Users\Administrator>curl -H "Cookie: Webstorm-be5713b6=883f4ca4-eb1c-4561-addb-a3f355dee357; jenkins-timestamper-offset=-28800000; JSESSIONID=849FE327C335DE3C4CFEA3D0ACF60E8C" -i http://localhost:8080
HTTP/1.1 200
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: text/plain;charset=UTF-8
Content-Length: 22
Date: Tue, 08 Dec 2020 06:05:20 GMT

hello, spring security


curl -H "Cookie: Webstorm-be5713b6=883f4ca4-eb1c-4561-addb-a3f355dee357; jenkins-timestamper-offset=-28800000; JSESSIONID=37C838C7BE7BBD40CB4A13EF0B0DCDE0" -i http://localhost:8080

#权限来自：cookie 中的JSESSIONID哈

注意有时报401错误 是访问的http方法不对 比如：POST、get 方式

#最基本的权限控制 http auth basic
spring.security.user.name=user  <br>  spring.security.user.password=123



@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                //表单登录方式 哈
                .formLogin()
                .loginPage("/myLogin.html")    // 前端页面
                .loginProcessingUrl("/login")  // 后端验证接口
                //.successHandler(new SecurityAuthenticationSuccessHandler()) //登录成功处理逻辑 可以处理数据等
                //.failureHandler(new SecurityAuthenticationFailureHandler())  //登录失败处理逻辑
                .defaultSuccessUrl("/hello",true)  //通常把true带上哈
                 // 使登录页不设限访问   一次次的上下文返回 连接操作
                .permitAll()
                .and()  //and 为终端符号
                .csrf().disable(); //跨域访问不允许
    }

}

而移动端直接访问 模拟form表单 访问后端就可以了

http://localhost:8081/login


C:\Users\Administrator>curl -X POST -H "Authorization:Basic dXNlcjoxMjM=" -i http://localhost:8081 <br>
#HTTP/1.1 302  302时临时重定向<br>
Set-Cookie: JSESSIONID=2039D7339A1F5EC3846881F2010FF212; Path=/; HttpOnly <br>
X-Content-Type-Options: nosniff <br>
X-XSS-Protection: 1; mode=block <br>
Cache-Control: no-cache, no-store, max-age=0, must-revalidate <br>
Pragma: no-cache  <br>
Expires: 0  <br>
X-Frame-Options: DENY  <br>
#Location: http://localhost:8081/myLogin.html    需要关注 Location地址 <br>
Content-Length: 0   返回302时已不关心返回的body了 看到内容为0哈 <br>
Date: Tue, 08 Dec 2020 06:46:25 GMT  <br>


这娃的例子里面尽然少了

response.setContentType("application/json;charset=UTF-8"); <br>
        PrintWriter out = response.getWriter(); <br>
        out.write("{\"error_code\":\"0\", \"message\":\"欢迎登录系统\"}"); <br>
out.flush(); <br>
out.close();

//可设置返回状态
@ResponseStatus(HttpStatus.NOT_MODIFIED)


//模拟表单的一次登录请求，返回了302  同时返回了set-Cookie  JSESSIONID=246CF1875714380D9C05D6C3654D960A
curl  -X POST -d "username=user&password=123"  -i http://localhost:8081/login

curl -X POST -H "Cookie: JSESSIONID=246CF1875714380D9C05D6C3654D960A"  http://localhost:8081/hello


curl -X POST -H "Cookie: JSESSIONID=C00FA677F39B78B266B697AAF36181E5"  http://localhost:8081/hello

#@RequestHeader("X-VOD-TIMESTAMP")  注解的使用哈

public ResponseEntity<Map<String, Object>> callback(@RequestBody(required = false) String callbackMessage,HttpServletRequest request,
														@RequestHeader("X-VOD-TIMESTAMP") String vodTimestamp,
														@RequestHeader("X-VOD-SIGNATURE") String vodSignature) throws Exception {



// 统一处理ajax未登录请求跳转到登录页面
$(document).ajaxError(function(event, request, settings) {
	if (request.responseText) {
		var data = $.parseJSON(request.responseText);
		if (data.status && data.status == '403') {
			window.location.href = '/mms/html/login.html';
		}
	}
});



//----------------add yuxh on 2017-6-18 增加一、二级域名的cookeie共用一个的特殊处理
		String  url=request.getHeader("Origin") ;
		Cookie cookieJSESSIONID = new Cookie("JSESSIONID", request.getSession().getId());
		//cookieJSESSIONID.setHttpOnly(false);
		//cookieJSESSIONID.setMaxAge(36000);
		cookieJSESSIONID.setPath("/");
		if (url!=null && url.indexOf("umosoft") >=0) {
			cookieJSESSIONID.setDomain("umosoft.com"); //localhost   umosoft.com
		}else {
			cookieJSESSIONID.setDomain("localhost");
		}	
		response.addCookie(cookieJSESSIONID);
		//-----------------------------end yuxh on 2017-6-17


请求转发和重定向的区别：

#请求转发是一个请求一次响应，而重定向是两次请求两次响应。 <br>
#请求转发地址不变化，而重定向会显示后一个请求的地址   <br>
#请求转发只能转发到本项目其它Servlet，而重定向不只能重定向到本项目的其它Servlet，还能定向到其它项目
请求转发是服务端行为，只需给出转发的Servlet路径，而重定向需要给出requestURI，既包含项目名

#这是请求转发  访问项目内的其他链接地址
request.getRequestDispatcher("/loginSuccessAjax").forward(request, response);

#前端直接走 取出链接直接重定向
if (res.data.successUrl) {
	window.location.href = res.data.successUrl;
} else {
	window.location.href = '/mms/index.html';
}


覆写这个 FormAuthenticationFilter  东东 控制 request、response


ShiroFilterFactoryBean 这个内配置 这个 FormAuthenticationFilter

controller 内再从  FormAuthenticationFilter内获取必要信息 之后，走  request.getRequestDispatcher 。。完成这一动作

可看出也是走的 必要的filter 完成的

org.apache.shiro.web.util.WebUtils


#shiro框架


路由拦截器

已路由为导向

router.beforeEach((to,from,next)=>{
  if(to.path=='/login' || localStorage.getItem('token')){
   next();
  }else{
   alert('请重新登录');
   next('/login');
  }
})
请求拦截器

当发送请求时才会触发此功能

axios.interceptors.request.use(function (config) {
 let token = window.localStorage.getItem("token");
   if (token) {
     config.headers.token = token;  //将token放到请求头发送给服务器
   }
   return config; // 最终需要返回config
  }, function (error) {
    return Promise.reject(error);
});



拦截响应
 就去请求到数据了，做一些数据判断，比如没有注册之类的，可以跳转到用户注册页面。

也可以判断请求是的token 是否过期，给它重置

// 拦截响应
http.interceptors.response.use(res => {
  // 响应失败
  if (!res.data.success) {
    Toast(res.data.msg)
    Indicator.close()
  }

  /**
   * refresh_token过期
   * 1、清空本地token
   * 2、刷新页面
   */
  if (res.data.code === '004-1') {
    localStorage.setItem('TOKEN', '')
    window.location.reload()
  }

  // 请先绑定手机号
  if (res.data.code === '004-2') {
    router.push({
      name: 'bindMobile'
    })
  }

  return res.data
}, error => {
  Toast(error.message)
  Indicator.close()
})


chcp  65001

.loginPage("/login401")    // 前端页面 login401   myLogin.html  浏览器与移动端复用这个接口工程

401是认证失败、403是授权失败

#CSRF全拼为Cross Site Request Forgery,跨站请求伪造

#

#1)实现接口 UserDetails
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.List;


public class User implements UserDetails {//这个是个好的思路  实现那部分哈

#2)mapper数据库落地
import com.blurooo.chapter3_2.entity.User;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Component;

@Component
public interface UserMapper {

    @Select("SELECT * FROM users WHERE username=#{username}")
    User findByUserName(@Param("username") String username);

}

#3）实现接口 UserDetailsService  奇怪呀只能走数据库方式
@Service
public class MyUserDetailsService implements UserDetailsService {//主要放入用户及角色

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {//实现这个就可以了
        // 从数据库尝试读取该用户
        User user = userMapper.findByUserName(username);
        // 用户不存在，抛出异常
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        // 将数据库形式的roles解析为UserDetails的权限集
        // AuthorityUtils.commaSeparatedStringToAuthorityList是Spring Security
        //提供的用于将逗号隔开的权限集字符串切割成可用权限对象列表的方法
        // 当然也可以自己实现，如用分号来隔开等，参考generateAuthorities
        user.setAuthorities(AuthorityUtils.commaSeparatedStringToAuthorityList(user.getRoles()));//加入角色部分
        return user;
    }
}



@EnableWebSecurity(debug = true)

---------------------------
自定义 核查思路

#1）增加验证码的逻辑存储
import org.springframework.security.authentication.AuthenticationDetailsSource;
import org.springframework.security.web.authentication.WebAuthenticationDetails;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;

@Component
public class MyWebAuthenticationDetailsSource implements AuthenticationDetailsSource<HttpServletRequest, WebAuthenticationDetails> {

    @Override
    public WebAuthenticationDetails buildDetails(HttpServletRequest request) {
        return new MyWebAuthenticationDetails(request);
    }

}

#2验证码 WebAuthenticationDetails 接收

import org.springframework.security.web.authentication.WebAuthenticationDetails;
import org.springframework.util.StringUtils;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

public class MyWebAuthenticationDetails extends WebAuthenticationDetails {

    private String imageCode;

    private String savedImageCode;

    public String getImageCode() {
        return imageCode;
    }

    public String getSavedImageCode() {
        return savedImageCode;
    }

    // 补充用户提交的验证码和session保存的验证码
    public MyWebAuthenticationDetails(HttpServletRequest request) {
        super(request);
        this.imageCode = request.getParameter("captcha");
        HttpSession session = request.getSession();
        this.savedImageCode = (String) session.getAttribute("captcha");
        if (!StringUtils.isEmpty(this.savedImageCode)) {
            // 随手清除验证码，不管是失败还是成功，所以客户端应在登录失败时刷新验证码
            session.removeAttribute("captcha");
        }
    }

}

#3）验证过程 DaoAuthenticationProvider

import com.blurooo.chapter4_2.exception.VerificationCodeException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

@Component
public class MyAuthenticationProvider extends DaoAuthenticationProvider {

    // 构造方法注入UserDetailService和PasswordEncoder
    public MyAuthenticationProvider(UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
        this.setUserDetailsService(userDetailsService);
        this.setPasswordEncoder(passwordEncoder);
    }

    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken) throws AuthenticationException {
        MyWebAuthenticationDetails details = (MyWebAuthenticationDetails) usernamePasswordAuthenticationToken.getDetails();//之前传入的映射过来哈
        String imageCode = details.getImageCode();
        String savedImageCode = details.getSavedImageCode();
        // 检验图形验证码
        if (StringUtils.isEmpty(imageCode) || StringUtils.isEmpty(savedImageCode) || !imageCode.equals(savedImageCode)) {
            throw new VerificationCodeException();
        }
        super.additionalAuthenticationChecks(userDetails, usernamePasswordAuthenticationToken);//这就是内部的核查逻辑
    }

}

----------------------------------------------------

图形验证码 开源的 jar ->  kaptcha  可引入此坐标
 
------------------------------
登录令牌的持久化

username: user
password: 123
remember-me: on

#1）持久化表

create table `persistent_logins` (
    username varchar(64) not null,
    series varchar(64) primary key, //获取加密用户
    token varchar(64) not null,     //获取用户
    last_used timestamp not null
);



import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;

import javax.sql.DataSource;

@EnableWebSecurity(debug = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Value("${spring.security.remember-me.key}")
    private String rememberKey;

    @Autowired
    private DataSource dataSource;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();

        tokenRepository.setDataSource(dataSource);

        http.authorizeRequests()
                .antMatchers("/admin/api/**")
                .hasRole("ADMIN")
                .antMatchers("/user/api/**")
                .hasRole("USER")
                .antMatchers("/app/api/**")
                .permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .permitAll()
                .and()
                .rememberMe()//记住我 存入数据库
                .userDetailsService(userDetailsService) //查找用户
                // 1. 散列加密方案
                .key(rememberKey)  //加密方式
                // 2. 持久化令牌方案
                .tokenRepository(tokenRepository)  //令牌存储方式
                // 7天有效期
                .tokenValiditySeconds(60 * 60 * 24 * 7) //过期时间
                .and()
                .logout() //退出登录状态
                //.logoutUrl("/myLogout")
                // 注销成功，重定向到该路径下
                .logoutSuccessUrl("/")
                .invalidateHttpSession(true)
                .and()
                .csrf()
                .disable();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}


链接不对产生多条数据  token数据


#获取登录用户信息
import com.blurooo.chapter5.entity.User;
import org.springframework.security.core.Authentication;

public interface IAuthenticationFacade {
    Authentication getAuthentication();
    User getUser();
}



import com.blurooo.chapter5.entity.User;
import com.blurooo.chapter5.service.IAuthenticationFacade;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;

@Component
public class AuthenticationFacade implements IAuthenticationFacade {

    @Override
    public Authentication getAuthentication() {
        return SecurityContextHolder.getContext().getAuthentication();
    }

    @Override
    public  User getUser() {
        User user = (User) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        return user ;
    }
}


spring.security.remember-me.key=blurooo

spring boot 配置session时间  spring.session.timeout=


jsonp 支持跨域，但只支持get请求哈

cors 跨域处理，允许跨域的域名

and().exceptionHandling().accessDeniedPage("/403");  // 处理异常，拒绝访问就重定向到 403 页面



#采用加密方式

 @Bean
    public BCryptPasswordEncoder bcryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }


    @Bean
    MyUserDetailsService MyUserDetailsService() {
        return new MyUserDetailsService();
    }

/**
 * 核心配置
 * @param auth
 * @throws Exception
 */
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
    authenticationProvider.setUserDetailsService(MyUserDetailsService());//内部使用
    authenticationProvider.setPasswordEncoder(bcryptPasswordEncoder());//内部使用不一样
    auth.authenticationProvider(authenticationProvider);
}

-------------------------------------------------------------------

#403的错误统一处理返回 状态码

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        response.setContentType(MediaType.APPLICATION_JSON_UTF8_VALUE);
        response.setStatus(HttpStatus.FORBIDDEN.value());
        response.getWriter().write(new ObjectMapper().writeValueAsString(new CustomResponse("无权访问", 403)));
    }

    static class CustomResponse {
        private int status;
        private String message;

        CustomResponse(String message, int Status) {
            this.message = message;
            this.status = Status;
        }

        public String getMessage() {
            return message;
        }

        public void setMessage(String message) {
            this.message = message;
        }

        public int getStatus() {
            return status;
        }

        public void setStatus(int status) {
            this.status = status;
        }
    }
}


.exceptionHandling().accessDeniedHandler(accessDeniedHandler()); 

//---------------------------------------------------

阿里的nacos  注册发现中心