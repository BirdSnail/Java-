# 1. 登录模块 shiro

用户认证：login

用户授权：什么人可以干什么



## 1.1 用户认证

- subject：主题，可以理解为用户

- principal：身份信息
- credential：凭证，如密码、证书、指纹

主题在用户认证时需要提供身份信息和凭证信息。

### 认证三大步

1. 收集用户的用户名，密码
2. 提交用户名，密码给认证系统进行认证
3. 允许访问，重新认证，拒绝访问

## 1.2 魔改shiro

**不同的`HTTP`方法使用不同的`Filter`**

1. 定义的URL要加入http method

2. 自定义我们的Filter

3. 修改`PathMatchingFilterChainResolver`的`getChain`逻辑（获取Filter）

   1. 原有的逻辑只判断URL路径，不会进行http method判断。

   2. 当URL匹配成功时，就会走我们定义的该URL对应的拦截器

      ```java
      @Bean
      public ShiroFilterFactoryBean shiroFilterFactoryBean(
          				SecurityManager securityManager) {
          final ShiroFilterFactoryBean shiroFilterFactory = 
              new HttpMethodShiroFilterFactoryBean();
          shiroFilterFactory.setSecurityManager(securityManager);
      
          LinkedHashMap<String, String> shiroFilterDefinitionMap = 
              new LinkedHashMap<>();
          shiroFilterDefinitionMap.put("/v1.0/session", "anon");
          shiroFilterDefinitionMap.put("/v1.0/users", "anon");
          shiroFilterDefinitionMap.put("/**", "authc");
          shiroFilterFactory.setFilterChainDefinitionMap(shiroFilterDefinitionMap);
      
          return shiroFilterFactory;
      }
      ```

   3. 修改后为：当URL和http method同时匹配成功时才会走我们定义的拦截器

## 1.3 自定义Shiro访问被拒绝时的返回状态

Shiro会拦截没有权限的请求，跳转到`/login.jsp`.如果我们是前后端分离的项目并不需要前端页面展示，因此需要我们自定义返回的内容。

当需要认证时，使用的拦截器为`FormAuthenticationFilter`,重写里面的`onAccessDenied()`

![image-20200410101149969](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200410101149969.png)



# 2. 测试

## 测试的三A原则

1. Arrange：准备好测试类以及所依赖的类，方法

2. Act：调用要测试的方法

3. Assert：断言返回的结果（或者`verify`方法的调用次数）

   1. 可以使用第三方框架：`hamcrest`

   ```xml
   <dependency>
       <groupId>org.hamcrest</groupId>
       <artifactId>hamcrest-all</artifactId>
       <version>1.3</version>
       <scope>test</scope>
   </dependency>
   ```

   

## Mockito

对于manager，dao层可以直接使用`Mockito`框架对需要注入的类进行mock，然后直接 `new`出我们的测试对象

```java
class UserManagerTest{

    private UserManager userManager;
    @Mock
    private UserDao userDao;

    @Test
    void setup(){
        userManager = new UserManager(userDao)
    }

}
```



## MockMvc

对于测试controller层，我们需要知道返回的状态值，http 头部信息，body等内容，因此要使用MockMvc，不能直接

```java
@ExtendWith(MockitoExtension.class)
class UserInfoControllerTest {

    private MockMvc mockMvc;
    @Mock
    private UserInfoManager userInfoManager;

    @BeforeEach
    void setup() {
        mockMvc = MockMvcBuilders.standaloneSetup(
            new	UserInfoController(userInfoManager))
            .setControllerAdvice(new GlobalExceptionHandler())
            .build();
    }
}
```



# 3. 系统设计

先围绕核心功能

## 3.1 要考虑到的问题

### 用户用例

- 专业用户用例图

### 解决问题领域

- 金融领域
- 财务领域
- 视频领域
- 。。。

### 解决问题范围

#### 功能边界

- 能不能购物
- 能不能交友
- 能不能推荐广告
- 。。。

#### 用户规模边界

- 用户规模 ，1亿用户 vs 100用户
- 用户在线时长
- 流量分布情况

## 3.2 需要哪些组件（服务）

**罗列所有的功能。选出核心功能**

#### 登录模块

- 用户登录（功能）
- 用户注册
- 用户信息修改
- 用户信息展示

贫血模型 vs 充血模型

#### 记账业务模块

- 记录账单交易
- 编辑账单交易
  - 单条编辑
  - 批量编辑
- 删除账单交易
  - 单挑删除
  - 批量删除
- 查询账单记录
- 按条件查询账单记录（会不会设计到分页）
  - 按日期查询
  - 按标签查询
  - 按收入/支出查询
  - 支持多个tag查询
  - 排序/分组
  - 分页

#### 标签模块

- 新增标签
- 编辑已有标签
- 删除已有标签

## 

## 3.3 数据的存储，访问

#### SQL V.S NoSQL

- 是否支持事务：SQL能够达到强一致性，NoSQL实现起来比较麻烦
- 是否有丰富的Query Builder：传统的Sql查询更加的有优势
- 对TPS要求高？是否要求很好的拓展性？
  - MySQL单机并发低。
  - NoSQL的并发更高，天生分布式，更容易扩展。

#### 表的细化

- 使用中间表

## 3.4 可用性

## 3.5 安全性

## 3.6 监控报警

# 4. 数据模型

跟业务紧密相关，定义良好的数据模型是业务开发的关键。

## 4.1 用户模型

```java
class UserInfo{
	Long id; 				// userId,主键，自增
	String userName;		// 用户名
    String passWord; 		// 密码
    String slat; 			// 密码加密用的盐
    LocalDateTime createTime; // 创建时间
    LocalDateTime updateTime; // 更新时间
}
```



## 4.2 标签模型

```java
class Tag{
	Long id; 				// tagId,主键，自增
    Long userId;		 	// 所属用户的userId
    Integer status; 		// 数据状态， 0 -> disable, 1 -> enable
    String description; 	// 标签描述（内容）
    LocalDateTime createTime; // 创建时间
    LocalDateTime updateTime; // 更新时间
}
```



## 4.3 Record模型

```java
class Record {
    Long id; 			// 主键，自增
    Long userId; 		// 用户id(这条记录所属用户)
    Integer status; 	// 数据状态，  0 -> disable, 1 -> enable
    BigDecimal amount;  // 金额
    Integer category;	// 类别，收入/支出
    String note; 		// 备注
    List<Tag> tagList;	// 包含的标签
    LocalDateTime createTime; // 创建时间
    LocalDateTime updateTime; // 更新时间
}
```



# 5. 疑难问题

- 模型转换

  - View层model没有的属性在DAO层进行设值。

- tag中间表

  ```tex
  record 与 tag 之间的映射关系。避免重复的存储。
    1. 一条record拥有多个tag。
    2. 所有的tag专门存储在一张表中。
    3. 根据一个record id可以映射出多个 tag id，然后根据tag id在tag表中查询实际的tag内容。
  ```

- 使用父类的equals and hascode

- 将自增的id插入数据库后写入原有的对象。

  - 使用Mybatis注解：`@Options`

- 查询结果状态List中。

- 更新接口的开发：

  - 全量更新：简单，但是会造成数据库的频繁访问。
  - 局部更新：判断原纪录与更新的记录的diff，只更新改变的字段，如果要更新`Tag`就涉及到中间表的更新，实现起来麻烦。
    1. tag 是否需要更新。
    2. 是的话 `disable/delete`原有的tag中间表（id, recordId, tagId）.
    3. 插入新的映射关系。

