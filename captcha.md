# 本地编译或者引入

## 1，maven 中央仓库引入jar包

`<!-- https://mvnrepository.com/artifact/com.github.ducheng/captcha-spring-boot-starter -->
<dependency>
    <groupId>com.github.ducheng</groupId>
    <artifactId>captcha-spring-boot-starter</artifactId>
    <version>0.0.3</version>
</dependency>`

## 2， 下载到本地编译成jar包

git clone https://github.com/ducheng/captcha-spring-boot-starter.git 

cd captcha-spring-boot-starter 

mvn clean install -DskipTests 

要打包跳过测试

# 使用

## 1，使用测试类

启动一个web的接口，注意不能和@RestController 一起使用

```java
@GetMapping（“ / index”）
@ReturnCaptcha（codeNumber = 6，disturbLinesize = 60）
public Captcha getindex（）{
    return Captcha.LINE;
}
```



## 2，注解解释

Captcha 是一个枚举，有三种 可以 选择验证码的干扰方式 LineCaptcha 线段干扰 (line) CircleCaptcha 圆圈干扰(circle)，ShearCaptcha 扭曲干扰(shear) 对应的四个属性 // 长、 int lengSize() default 200;

```java
// 宽、
int widhSize() default 100;

// 验证码字符数、
int codeNumber() default 4;

// 干扰线宽度
int disturbLinesize() default 4;
```

## 3，代码原理

### 核心代码 

RetuenCaptchReturnValueHandler 类， 实现了返回参数处理器

```java
public class RetuenCaptchReturnValueHandler implements HandlerMethodReturnValueHandler {

private static final Logger LOGGER = LoggerFactory.getLogger(RetuenCaptchReturnValueHandler.class);

@Override
public boolean supportsReturnType(MethodParameter returnType) {
	// 判断是否存在这个注解， 这是自己写的自定义的注解 ,只有当结果为true 才执行下面的方法
	return returnType.hasMethodAnnotation(ReturnCaptcha.class);
}

@Override
public void handleReturnValue(Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer,
		NativeWebRequest webRequest) throws Exception {
	/* check */
	HttpServletResponse response = (HttpServletResponse) webRequest.getNativeResponse(HttpServletResponse.class);
	// 断言判断是否存在返回值
	Assert.state(response != null, "No HttpServletResponse");
	//获取方法上面的注解
	ReturnCaptcha returnCaptcha = returnType.getMethodAnnotation(ReturnCaptcha.class);
	Assert.state(returnCaptcha != null, "No @ReturnCaptcha");
	mavContainer.setRequestHandled(true);
	// 判断返回类型
	if (!(returnValue instanceof Captcha)) {
		throw new Exception(" 返回类型 不匹配");
	}else { 
	// 根据注解里面的参数返回不同的流
		AbstractCaptcha abstractCaptcha = null;
		Captcha captcha =(Captcha)returnValue;
		switch (captcha) {
		// 支持三种验证码格式
		case LINE:
			abstractCaptcha = CaptchaUtil.createLineCaptcha(returnCaptcha.lengSize(),
					returnCaptcha.widhSize(),returnCaptcha.codeNumber(),returnCaptcha.disturbLinesize());
			abstractCaptcha.write(response.getOutputStream());
        case SHEAR:
        	abstractCaptcha = CaptchaUtil.createShearCaptcha(returnCaptcha.lengSize(),
					returnCaptcha.widhSize(),returnCaptcha.codeNumber(),returnCaptcha.disturbLinesize());
			abstractCaptcha.write(response.getOutputStream());
		default:
			abstractCaptcha = CaptchaUtil.createCircleCaptcha(returnCaptcha.lengSize(),
					returnCaptcha.widhSize(),returnCaptcha.codeNumber(),returnCaptcha.disturbLinesize());
			abstractCaptcha.write(response.getOutputStream());
		}
	}
    //最后别忘记一定要关流
	response.getOutputStream().flush();
	response.getOutputStream().close();
}}
```
### 将其加入 webMVC配置 里面

```java
public class CaptchaWebMvcConfiguation implements WebMvcConfigurer {

	private static final Logger LOGGER = LoggerFactory.getLogger(CaptchaWebMvcConfiguation.class);
	
	
	//默认是开启状态
	private boolean enable = true;
	
	public boolean isEnable() {
		return enable;
	}
	
	public void setEnable(boolean enable) {
		this.enable = enable;
	}
	
	@Override
	public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {
		if (this.enable) {
			LOGGER.info(" 注入 验证码web 容器成功 ");
			handlers.add(new RetuenCaptchReturnValueHandler());
		}
	}}
```



### 在使用application.properties  文件配置开启

```java
@Configuration
@EnableConfigurationProperties(CaptchaAutoConfigure.class)
@ConfigurationProperties("com.ducheng.captcha")
public class CaptchaAutoConfigure {
	
	private static final Logger LOGGER = LoggerFactory.getLogger(CaptchaAutoConfigure.class);
	
	private boolean enable = true;

	public boolean isEnable() {
		return enable;
	}
	public void setEnable(boolean enable) {
		this.enable = enable;
	}
	@Bean
	public WebMvcConfigurer excelWebMvcConfigurer() {
		CaptchaWebMvcConfiguation configurer = new CaptchaWebMvcConfiguation();
		configurer.setEnable(this.enable);
		LOGGER.info("初始化 验证码 web 容器");
		return configurer;
	}
}
```

在application 默认是开启的

com.ducheng.captcha =false 可以关闭

### spring.factories添加

在[resources](https://github.com/ducheng/captcha-spring-boot-starter/tree/master/src/main/resources)/[META-INF](https://github.com/ducheng/captcha-spring-boot-starter/tree/master/src/main/resources/META-INF)/spring.factories

里面添加 org.springframework.boot.autoconfigure.EnableAutoConfiguration=\

com.ducheng.springboot.aotoconfigure.CaptchaAutoConfigure