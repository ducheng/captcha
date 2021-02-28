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

`@GetMapping("/index") 

@ReturnCaptcha(codeNumber = 6,disturbLinesize = 60) 

public Captcha getindex() {

 return Captcha.LINE; }`

## 2，注解解释

Captcha 是一个枚举，有三种 可以 选择验证码的干扰方式 LineCaptcha 线段干扰 (line) CircleCaptcha 圆圈干扰(circle)，ShearCaptcha 扭曲干扰(shear) 对应的四个属性 // 长、 int lengSize() default 200;

```
// 宽、
int widhSize() default 100;

// 验证码字符数、
int codeNumber() default 4;

// 干扰线宽度
int disturbLinesize() default 4;
```

## 3，代码原理

## 