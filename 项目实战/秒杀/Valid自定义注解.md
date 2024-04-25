# Valid自定义注解
1、引入依赖
2、定义interface
注意
@Constraint(validatedBy = { isMobileValidator.class})
```
package com.wpp.skill.validator;

import com.wpp.skill.vo.isMobileValidator;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;


/**
 * 手机号码判断是否正确工具类
 */
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = { isMobileValidator.class})
public @interface IsMobile {

    boolean required() default true;
    String message() default "手机号格式错误";
    Class<?>[] groups() default { };
    Class<? extends Payload>[] payload() default { };
}

```
3、定义接口的检验方法
注意泛型IsMobile
```
package com.wpp.skill.vo;

import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.wpp.skill.utils.ValidatorUtils;
import com.wpp.skill.validator.IsMobile;


import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

public class isMobileValidator implements ConstraintValidator<IsMobile,String> {
    private boolean required = false;
    @Override
    public void initialize(IsMobile constraintAnnotation) {
        required = constraintAnnotation.required();
    }

    @Override
    public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {
        if(required){
            return ValidatorUtils.validMobile(s);
        }else {
            if(StringUtils.isBlank(s)){
                return true;
            }else {
                return ValidatorUtils.validMobile(s);
            }
        }
    }
}

```
4、应用
在传入参数中加注解@Vaild
```
public ResBean doLogin(@Valid LoginVo loginVo)
```
在实体类中加接口注解
```
@Data
public class LoginVo {
    @NotNull
    @IsMobile
    private String mobile;

    @NotNull
    @Length(min=32)
    private String password;
}

```
最终会在日志中体现
Field error in object 'loginVo' on field 'mobile': rejected value [157084358571111]; codes [IsMobile.loginVo.mobile,IsMobile.mobile,IsMobile.java.lang.String,IsMobile]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [loginVo.mobile,mobile]; arguments []; default message [mobile],true]; default message [手机号格式错误]]

5、所以要进行异常处理
所用注解：
@ExceptionHandler：统一处理某一类异常，从而能够减少代码重复率和复杂度
@ControllerAdvice：异常集中处理，更好的使业务逻辑与异常处理剥离开；其是对Controller层进行拦截
@ResponseStatus：可以将某种异常映射为HTTP状态码