# portal-oauth-platform
门户单点登录配置化平台，前后端代码（vue+springboot+mongo）

## 概述
之前已经通过本系列两篇文章[《单点登录系列（1）- OAuth2实施方案 》](http://paperbee.definesys.com/content?id=0ed3b1e8ae3311e991cefa82b547b9ed)、[《单点登录系列（2）- Spring OAuth2项目搭建》](http://paperbee.definesys.com/content?id=8f0c1e22b10b11e9817afa82b547b9ed)，跟大家介绍了OAuth2的单点登录方案，以及如何用Spring搭建OAuth2项目。对于有些朋友来说，单点登录的这个概念还是比较抽象。

我想今天这篇文章，配合文中的一个配置OAuth2的Web平台，争取让不会代码的人也能轻松理解OAuth2的单点登录。包括这次把上篇文章中代码重构了一下，优化了很多功能，本文也会单独做说明。


## OAuth2 Web配置平台
网址是：[http://kerrysmec.cn:81](http://kerrysmec.cn:81)。做这个web平台的目的，很大程度是想玩玩vue和移动适配。

平台本次做了三个配置的模块：客户端、登录页和用户账号配置。
1. 客户端配置：第三方平台地址接入OAuth2的配置。
2. 登录页配置：切换自定义登录页。
3. 账号配置：登录账号密码管理。

### 1.客户端配置
回顾一下《单点登录系列（1）- OAuth2实施方案 》中，每个第三方平台如果想接入OAuth2的单点登录，都需要申请client_id和client_secret。那么我们就将这个第三方平台称为客户端，下面这个界面就是配置新客户端clientId等参数的。

![avatar](http://paperbee.definesys.com/file/img/75dd91fed1124bc4b4ffa27d45060f4f.png)

介绍一下需求配置客户端的几个参数：
1. 客户端编码：client_id，业务主键
2. 客户端密钥：client_secret，密钥
3. 授权范围：只选取了常用的“授权码模式”和“密码模式”，authorization_code可以通过“授权码模式”调整登录页；password可以通过“密码模式”调接口获取用户token信息；refresh_token则可以通过refresh_token获取最新的access_token。
4. 重定向url：是配置用户在登录页登录成功后，重定向跳转到该客户端的页面地址。这里值得注意的是：（1）如果配置了“重定向url”，单点登录就只能重定向到该地址；（2）如果没有配置“重定向url”，该字段为空，则可以重定向到自定义拼接的任意地址。
5.Access Token有效期：因为我配置的是Jwt token，有时间限制，单位为“秒”，如图中access token在7200秒后过期。
6.Refresh Token有效期：同理，259200秒后失效。

如图中，我们新增了四行数据，即配置了四个接入单点登录的客户端，第一个重定向地址为空，后面三个分别是得帆、新浪、腾讯、百度。

![avatar](http://paperbee.definesys.com/file/img/00f175f5ff1246d3acf0dc46cc8f8067.png)

那么该如果访问客户端接入单点登录的网址呢？客户端表格中操作列的第一个按钮，点击打开客户端详情弹框，我们打开baiduClient的弹框，点击“访问”按钮，或者复制单点登录的访问地址到浏览器就可以啦。

不过这时我们就会发现，无论我们访问这四个客户端里面的那一个，都会跳转到我们的登录页。说明这些第三方的平台已经受我们单点登录的控制了，我们可以试一下，在成功登录任意一个客户端后，再点击任意平台的“访问”按钮，就可以直接进入对应的重定向网址，而不会再被拦截在登录页了。（登录的账号密码在后文“账号配置”中设置）

大家细心点会发现，在成功登录进入重定向的页面时，重定向url后面跟了一个参数code。在密码模式里面就可以通过调接口，通过这个code的值，获取包含当前用户账号信息的access_token和refresh_token，具体细节就参考本系列第一篇文章，不过我们可以用到上述弹框中的参数 Basic Authoriztion 。

如果客户端中没有配置重定向url，根据前文说的，我们可以在访问地址中写上任何网址，不受拘束。

![avatar](http://paperbee.definesys.com/file/img/17b79c3d995040b6bbeb98758db3a805.png)

![avatar](http://paperbee.definesys.com/file/img/9d0500fc56cd4b9eaa72893fd7c84925.png)


### 2.登录页选择
我喜欢多姿多彩的登录页，甚至于像谷歌搜索的主页一样，一年遇到不同的节日，都有不同的页面风格切换。

没错，这个登录页配置功能，就是让你可以实时的切换单点登录系统的登录页。在你喜欢的登录页卡片上，点击“选用”就生效了!我这里偷懒，没有设计很多登录页，只是弄了几个不同色系的页面。

![avatar](http://paperbee.definesys.com/file/img/ce9007f3022749068dde8e2ec06d5253.png)

### 3.账号管理
既然是单点登录，用统一的登录页，那么账号密码也是唯一入口配置的。在企业里面员工的账号密码有AD、OID等等来管理。就算存在表里面，也是加密的方式，不会是像我开发的这样明文显示。

但是为了这个平台的功能完整，我还是加了这个最简单的账号/密码管理的页面，方便大家自行操作。创建好“用户名”和“密码”，然后就可以在登录页上成功登录了。

![avatar](http://paperbee.definesys.com/file/img/4fa8b650ab5d4866b8a2b464e7c41b04.png)

## Spring OAuth2 的补充
[《单点登录系列（2）- Spring OAuth2项目搭建》](http://paperbee.definesys.com/content?id=8f0c1e22b10b11e9817afa82b547b9ed) 中对于客户端的配置比较粗糙，是在项目启动时一次性读取所有客户端client的配置信息，放在内存里面实现的。如果有需要加接入新的客户端就需要重启后再生效，我们能不能把配置信息放在表里面呢？就像我们上面Web平台做的那样。

### 1.client 基于表
Spring OAuth2的官方框架里面就封装了表oauth_client_details，要求只要你按照他的要求建一个同样的表，将读取client的方式换成这种方式，就可以实现动态的从表里面读取client的配置信息。

AuthorizationServerConfigurerAdapter.java
``` java
@Resourceprivate DataSource dataSource; 
@Bean public ClientDetailsService clientDetailsService() {    
return new JdbcClientDetailsService(dataSource); 
} 

@Override public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {  
clients.withClientDetails(clientDetailsService());
 }
```
oauth_client_details 表结构

| 字段名                  | 字段描述                 | 详细描述                                                     | 范例                                                         |
| ----------------------- | ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| client_id               | 主键，必须唯一，不能为空 | 用于唯一标识每一个客户端(client)；注册时必须填写(也可以服务端自动生成)，这个字段是必须的，实际应用也有叫app_key | OaH1heR2E4eGnBr87Br8FHaUFrA2Q0kE8HqZgpdg8Sw                  |
| resource_ids            | 不能为空，用逗号分隔     | 客户端能访问的资源id集合，注册客户端时，根据实际需要可选择资源id，也可以根据不同的额注册流程，赋予对应的额资源id | order-resource,pay-resource                                  |
| client_secret           | 必须填写                 | 注册填写或者服务端自动生成，实际应用也有叫app_secret, 必须要有前缀代表加密方式 | {bcrypt}![07240e742f4facca14c99fe78c2cff2b.svg+xml](en-resource://database/1792:0)gY/Hauph1tqvVWiH4atxteSH8sRX03IDXRIQi03DVTFGzKfz8ZtGi |
| scope                   | 不能为空，用逗号分隔     | 指定client的权限范围，比如读写权限，比如移动端还是web端权限  | read,write / web,mobile                                      |
| authorized_grant_types  | 不能为空                 | 可选值 授权码模式:authorization_code,密码模式:password,刷新token: refresh_token, 隐式模式: implicit: 客户端模式: client_credentials。支持多个用逗号分隔 | password,refresh_token                                       |
| web_server_redirect_uri | 可为空                   | 客户端重定向uri，authorization_code和implicit需要该值进行校验，注册时填写， | http://baidu.com                                             |
| authorities             | 可为空                   | 指定用户的权限范围，如果授权的过程需要用户登陆，该字段不生效，implicit和client_credentials需要 | ROLE_ADMIN,ROLE_USER                                         |
| access_token_validity   | 可空                     | 设置access_token的有效时间(秒),默认(606012,12小时)           | 3600                                                         |
| refresh_token_validity  | 可空                     | 设置refresh_token有效期(秒)，默认(606024\*30, 30填)          | 7200                                                         |
| additional_information  | 可空                     | 值必须是json格式                                             | {&quot;key&quot;, &quot;value&quot;}                         |
| autoapprove             | false/true/read/write    | 默认false,适用于authorization_code模式,设置用户是否自动approval操作,设置true跳过用户确认授权操作页面，直接跳到redirect_uri | false                                                        |

### 2.client基于接口（ClientDetailsService）
第一种方式和框架绑的太死，我推荐用这种方式-基于 ClientDetailsService 接口来实现。我在mongo里面自己创建了一个表，不用要求完全和第一种方式的表结构一样，只要实现了该接口就行。

OauthClientServiceImpl.java
``` java
@Service
public class OauthClientServiceImpl implements ClientDetailsService {
    @Autowired
    BCryptPasswordEncoder bCryptPasswordEncoder;

    @Autowired
    private PaasDao paasDao;


    @Override
    public BaseClientDetails loadClientByClientId(String clientId)  {
        OauthClientEO client = paasDao.getClientByClientId(clientId);
        if (client == null) {
            throw new ClientRegistrationException(clientId+"无效") ;
        }
        BaseClientDetails baseClientDetails=new BaseClientDetails();
        baseClientDetails.setClientId(client.getClientId());
        baseClientDetails.setClientSecret(EncoderUtil.encryptByBCrypt(client.getClientSecret()));
        baseClientDetails.setAccessTokenValiditySeconds(client.getAccessTokenValidity());
        baseClientDetails.setRefreshTokenValiditySeconds(client.getRefreshTokenValidity());
        baseClientDetails.setAuthorizedGrantTypes(Arrays.asList(client.getAuthorizedGrantTypes().split(",")));
        Collection<SimpleGrantedAuthority> collection = new HashSet<>();
        collection.add(new SimpleGrantedAuthority("ROLE_USER"));
        baseClientDetails.setAuthorities(collection);
        Set<String> scope = new TreeSet<String>();
        scope.add("user_info");
        baseClientDetails.setScope(scope);
        baseClientDetails.setAutoApproveScopes(scope);
        return baseClientDetails;
    }
}

```
AuthorizationServerConfigurerAdapter.java
``` java
/**
 * 客户端验证，自定义实现oauthClientService接口
 */
 @Autowired
    private OauthClientServiceImpl oauthClientServiceImpl;

 @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.withClientDetails(oauthClientServiceImpl).clients(oauthClientServiceImpl);
    }
```

### 3.user基于接口（UserDetailsService）
和客户端client一样，账号/密码的校验也是可以通过实现接口来完成的。代码中 PasswordEncoderImpl( ) 是我自定义的加密类，因为企业中密码一般都是加密的，比如base64+md5，这时候就可以自定义去实现这个加密类里面的方法。

UserDetailsServiceImpl.java
``` java
@Component
public class UserDetailsServiceImpl implements UserDetailsService {
    @Autowired
    private PaasDao paasDao;

    @Override
    public UserDetails loadUserByUsername(String username) throws BadCredentialsException {
        //enable ：用户已失效
        //accountNonExpired：用户帐号已过期
        //credentialsNonExpired：坏的凭证
        //accountNonLocked：用户账号已锁定
        //String password= userFeign.loadUserByUsername(username);
        UserEO userEO=paasDao.getUserByUsername(username);
        return new User(username, new PasswordEncoderImpl().encode(userEO.getPassword()), AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_USER"));

    }
```