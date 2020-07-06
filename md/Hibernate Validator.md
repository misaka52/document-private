# Hibernate Validator

**By -李梦佳**



## What

hibernate-validator 是 hibernate 组织下的一个[开源项目 ](https://github.com/hibernate/hibernate-validator)。

hibernate-validator是 [Jakarta Bean Validation 3.0](http://beanvalidation.org/).规范的实现。

Jakarta Bean Validation 3.0定义了一个实体和方法验证的元数据模型和 API。

- 通过使用注解的方式在对象模型上表达约束
- 以扩展的方式编写自定义约束
- 提供了用于验证对象和对象图的API
- 提供了用于验证方法和构造方法的参数和返回值的API
- 报告违反约定的集合
- 运行在Java SE，并且集成在Jakarta EE8中

使用前：

![application layers](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/images/application-layers.png)

使用后：

![application layers2](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/images/application-layers2.png)

## 引用

```
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-validator</artifactId>
   <version>6.1.5.Final</version>
</dependency>
```

spring-boot-starter-web已集成，可以直接使用

## 注解

### 标识注解

### @Valid（javax）

可以用在方法、构造函数、方法参数和成员属性（字段）上。不提供分组方法。

```java
    @Valid
    @NotNull(message = "承保请求数据集不能为空")
    private List<PolicyReqDTO> policyReqList;
```

```java
 public ResponseEntity save(@Valid JdjrInsuredRequest dto, BindingResult bindingResult)
```



### @Validated（spring）

可以用在类型、方法和方法参数上。但是不能用在成员属性（字段）上。提供分组方法。

```java
 public ResponseEntity save(@Validated({YouzanParamsValidator.class}) JdjrInsuredRequest dto, BindingResult bindingResult)
```

区别：

1. 嵌套验证需要使用@Valid对成员
2. @Validated支持分组

### 约束注解

### @AssertFalse

检查元素是否为 false，支持数据类型：boolean、Boolean

### @AssertTrue

检查元素是否为 true，支持数据类型：boolean、Boolean

### @DecimalMax(value=, inclusive=)

inclusive：boolean，默认 true，表示是否包含，是否等于
value：当 inclusive=false 时，检查带注解的值是否小于指定的最大值。当 inclusive=true 检查该值是否小于或等于指定的最大值。参数值是根据 bigdecimal 字符串表示的最大值。
支持数据类型：BigDecimal、BigInteger、CharSequence、（byte、short、int、long 和其封装类）

### @DecimalMin(value=, inclusive=)

支持数据类型：BigDecimal、BigInteger、CharSequence、（byte、short、int、long 和其封装类）
inclusive：boolean，默认 true，表示是否包含，是否等于
value：
当 inclusive=false 时，检查带注解的值是否大于指定的最大值。当 inclusive=true 检查该值是否大于或等于指定的最大值。参数值是根据 bigdecimal 字符串表示的最小值。

### @Digits(integer=, fraction=)

检查值是否为最多包含 `integer` 位整数和 `fraction` 位小数的数字
支持的数据类型：
BigDecimal, BigInteger, CharSequence, byte, short, int, long 、原生类型的封装类、任何 Number 子类。

### @Email

检查指定的字符序列是否为有效的电子邮件地址。可选参数 `regexp` 和 `flags` 允许指定电子邮件必须匹配的附加正则表达式（包括正则表达式标志）。
支持的数据类型：CharSequence

### @Max(value=)

检查值是否小于或等于指定的最大值
支持的数据类型：
BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（**字符序列表示的数字**）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类

### @Min(value=)

检查值是否大于或等于指定的最大值
支持的数据类型：
BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（**字符序列表示的数字**）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类

### @NotBlank

检查字符序列是否为空，以及去空格后的长度是否大于 0。与 `@NotEmpty` 的不同之处在于，此约束只能应用于字符序列，并且忽略尾随空格。
支持数据类型：CharSequence

### @NotNull

检查值是否**不**为 `null`
支持数据类型：任何类型

### @NotEmpty

检查元素是否为 `null` 或 `空`
支持数据类型：CharSequence, Collection, Map, arrays

### @Size(min=, max=)

检查元素个数是否在 min（含）和 max（含）之间
支持数据类型：CharSequence，Collection，Map, arrays

### @Negative

检查元素是否严格为负数。零值被认为无效。
支持数据类型：
BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（**字符序列表示的数字**）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类

### @NegativeOrZero

检查元素是否为负或零。
支持数据类型：
BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（**字符序列表示的数字**）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类

### @Positive

检查元素是否严格为正。零值被视为无效。
支持数据类型：
BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（**字符序列表示的数字**）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类

### @PositiveOrZero

检查元素是否为正或零。
支持数据类型：
BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（**字符序列表示的数字**）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类

### @Null

检查值是否为 `null`
支持数据类型：任何类型

### @Future

检查日期是否在未来
支持的数据类型：
java.util.Date, java.util.Calendar, java.time.Instant, java.time.LocalDate, java.time.LocalDateTime, java.time.LocalTime, java.time.MonthDay, java.time.OffsetDateTime, java.time.OffsetTime, java.time.Year, java.time.YearMonth, java.time.ZonedDateTime, java.time.chrono.HijrahDate, java.time.chrono.JapaneseDate, java.time.chrono.MinguoDate, java.time.chrono.ThaiBuddhistDate
如果 [Joda Time](http://www.joda.org/joda-time/) API 在类路径中，`ReadablePartial` 和`ReadableInstant` 的任何实现类

### @FutureOrPresent

检查日期是现在或将来
支持数据类型：同@Future

### @Past

检查日期是否在过去
支持数据类型：同@Future

### @PastOrPresent

检查日期是否在过去或现在
支持数据类型：同@Future

### @Pattern(regex=, flags=)

根据给定的 `flag` 匹配，检查字符串是否与正则表达式 `regex` 匹配
支持数据类型：CharSequence

**......**

**所有约束注解一定有的三个属性：**

-  `message` 属性：在违反约束的情况下返回一个默认 key 以用于创建错误消息
-  `groups` 属性：允许指定此约束所属的验证分组。必须默认是一个空 Class 数组
-  `payload` 属性：能被 Bean Validation API 客户端使用，以自定义一个注解的 payload 对象。API 本身不使用此属性。



## 使用

### 实体类约束

1. 字段级别约束

   ```java
   	/**
        * 主商品sku名称
        */
       @NotBlank(message = "主商品sku名称不能为空")
       @Length(max = 1000, message = "主商品sku名称超长")
       private String mainSkuName;
   ```

   约束可以应用于任何访问类型（公共，私有等）的字段。不支持对静态字段的约束。

2. 属性级别约束

   ```java
   	@NotBlank
       public String getChannel(){
           return channel;
       }
   ```

   属性级别约束和字段级别约束推荐选一使用

3. 容器级别约束

   ```
   
   private Map<@Min(value = 0) Integer,@NotBlank String> productInfo;
   ```
```
   
Hibernate Validator验证以下标准Java容器上指定的容器元素约束：
   
   - `java.util.Iterable`的实现（例如`List`，`Set`）
   - `java.util.Map`的实现，支持键和值的验证
   - `java.util.Optional`，`java.util.OptionalInt`，`java.util.OptionalDouble`，`java.util.OptionalLong`，
- JavaFX的各种实现`javafx.beans.observable.ObservableValue`。
   
6.0之前的版本需要在容器上加@Valid，6.0之后的不需要。
   
4. 类级别约束

   在这种情况下，验证的对象不是单个属性，而是完整的对象。验证依赖于对象的多个属性之间的相关性，类级约束非常有用。

5. 约束继承

   当一个类实现接口或扩展另一个类时，在超类型上声明的所有约束注释都以与在该类本身上指定的约束相同的方式应用。如果方法被覆盖，约束注释将被聚合。

   校验时会取到该类的所有继承关系，循环遍历所有约束并校验。

6. 级联验证

```
   @Data
   public class LSInsuredRequestData {
       /**
        * 承保请求数据集
        */
       @Valid
       @NotNull(message = "承保请求数据集不能为空")
       private List<PolicyReqDTO> policyReqList;
   }

   public class PolicyReqDTO {
       /**
        * 延保单号
        */
       @NotBlank(message = "延保单号不为为空")
       private String orderNo;
       /**
        * 保单标识码
        */
       @NotBlank(message = "保单标识码不能为空")
       private String policyNo;
       /**
        * 主商品sku
        */
       @NotBlank(message = "主商品sku不能为空")
       private String mainSkuCode;
       /**
        * 主商品sku名称
        */
       @NotBlank(message = "主商品sku名称不能为空")
       @Length(max = 1000, message = "主商品sku名称超长")
       private String mainSkuName;
       /**
        * 唯一编码（IMEI号）
        */
       private String serialNo;

       /**
        * 保额
        */
       @Digits(integer=12, fraction=2, message = "保额格式错误")
       @Min(value = 0, message = "金额不能小于0")
       private BigDecimal policyCoverage;

   

   }
   ```

### 方法约束

1. 参数约束

通过向方法或构造函数的参数添加约束注解来指定方法或构造函数的前置条件

   ```
public void test(@NotBlank String policyNo){
    }
```

2. 返回值约束

通过在方法体上添加约束注解来给方法或构造函数指定后置条件

```
	@Min(value = 0)
	public int testOutput(int value){
	    return value;
	}
```

3. 级联约束

类似于 JavaBeans 属性的级联验证，`@Valid` 注解可用于标记方法参数和返回值的级联验证。

```
public void testList(@NotEmpty List<@Valid PolicyReqDTO> reqDTOList){
        
    }
```



### 分组约束

约束注解都有groups属性，默认未Default分组。

定义分组：

```
public interface YouzanParamsValidator extends Default {
}

```

使用分组：

​```java
/**
     *   用户电话
     */
    @NotBlank(message = "用户电话不能为空",groups = {JdParamsValidator.class})
    @Length(max = 256, message = "用户电话超长")
    private String phone;
    
    /**
     *  保单明细
     */
    @NotNull(message = "保单明细不能为空",groups = {YouzanParamsValidator.class})
    @Valid
    private List<PolicyAttachs> policyAttachs;
```

#### 分组序列 

场景：

当a=1时，校验b

当a=2时，校验c

例如选择必填等

`Hibernate Validator`提供了**非标准**的`@GroupSequenceProvider`注解。本功能提供根据当前对象实例的状态，**动态来决定**加载那些校验组进入默认校验组。

```java
// 该接口定义了：动态Group序列的协定
// 要想它生效，需要在T上标注@GroupSequenceProvider注解并且指定此类为处理类
// 如果`Default`组对T进行验证，则实际验证的实例将传递给此类以确定默认组序列
public interface DefaultGroupSequenceProvider<T> {
	List<Class<?>> getValidationGroups(T object);
}
```

```java
public class TestSequenceProvider implements DefaultGroupSequenceProvider<Insured> {

    @Override
    public List<Class<?>> getValidationGroups(Insured bean) {
        List<Class<?>> defaultGroupSequence = new ArrayList<>();
        defaultGroupSequence.add(Insured.class); // 这一步不能省,否则Default分组都不会执行

        if (bean != null) { 
            Integer channel = bean.getChannel();
            if (channel = 1) {
                defaultGroupSequence.add(Insured.JdParamsValidator.class);
            } else if (channel = 2) {
                defaultGroupSequence.add(Insured.YouzanParamsValidator.class);
            }
        }
        return defaultGroupSequence;
    }
}
```

```java
@GroupSequenceProvider(TestSequenceProvider.class)
@Getter
@Setter
@ToString
public class Insured {

    @NotNull
    private Integer channel;

    /**
     *   用户电话
     */
    @NotBlank(message = "用户电话不能为空",groups = {JdParamsValidator.class})
    @Length(max = 256, message = "用户电话超长")
    private String phone;
    
    /**
     *  保单明细
     */
    @NotNull(message = "保单明细不能为空",groups = {YouzanParamsValidator.class})
    @Valid
    private List<PolicyAttachs> policyAttachs;
}
```



### 自定义约束

1. 创建约束注解

```java
@Target({ TYPE , ANNOTATION_TYPE })
@Retention(RUNTIME)
@Constraint(validatedBy = { PremiumAmountValidator.class })
@Documented
public @interface PremiumAmountCheck {
    String message() default "保费保额校验错误";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    double rate();
}
```

`Bean Validation API` 规范要求任何约束注解定义以下属性：`message` 、`groups`、`payload`  

2. 实现验证器

   ```java
   public interface ConstraintValidator<A extends Annotation, T> {
   
   	/**
   	 * Initializes the validator in preparation for
   	 * {@link #isValid(Object, ConstraintValidatorContext)} calls.
   	 * The constraint annotation for a given constraint declaration
   	 * is passed.
   	 * <p>
   	 * This method is guaranteed to be called before any use of this instance for
   	 * validation.
   	 * <p>
   	 * The default implementation is a no-op.
   	 *
   	 * @param constraintAnnotation annotation instance for a given constraint declaration
   	 */
   	default void initialize(A constraintAnnotation) {
   	}
   
   	/**
   	 * Implements the validation logic.
   	 * The state of {@code value} must not be altered.
   	 * <p>
   	 * This method can be accessed concurrently, thread-safety must be ensured
   	 * by the implementation.
   	 *
   	 * @param value object to validate
   	 * @param context context in which the constraint is evaluated
   	 *
   	 * @return {@code false} if {@code value} does not pass the constraint
   	 */
   	boolean isValid(T value, ConstraintValidatorContext context);
   }
   ```

   ConstraintValidator` 指定了两个泛型类型：

   1. 第一个是指定需要验证的注解类
   2. 第二个是指定要验证的数据类型，当注解支持多种类型时，就要写多个实现类，并分别指定对应的类型

   需要实现两个方法：

   - `initialize()` 让你可以获取到使用注解时所指定的参数（default方法，可以不实现）
   - `isValid()` 包含实际的校验逻辑。

```java
//校验JdjrInsuredInfoDTO中保费=保额*rate
public class PremiumAmountValidator implements ConstraintValidator<PremiumAmountCheck, JdjrInsuredInfoDTO> {
    private double rate;
    @Override
    public void initialize(PremiumAmountCheck constraintAnnotation) {
        rate = constraintAnnotation.rate();
    }

    @Override
    public boolean isValid(JdjrInsuredInfoDTO jdjrInsuredInfoDTO, ConstraintValidatorContext constraintValidatorContext) {
        Boolean isValid = jdjrInsuredInfoDTO.getPremium().compareTo(NumberUtil.mul(jdjrInsuredInfoDTO.getAmount(),rate)) == 0;;
        if(!isValid){
            constraintValidatorContext.disableDefaultConstraintViolation();
            constraintValidatorContext.buildConstraintViolationWithTemplate(
                    "保费保额错误，{rate}，{message}"
            )
                    .addConstraintViolation();
        }
        return isValid;
    }

}

```

```
javax.validation.ConstraintViolationException: testOutput.value: 保费保额错误，0.5，保费保额校验错误
```

如果同一个约束可以应用于多种数据类型，需要分别实现每种数据类型的验证器

```java
public class NegativeOrZeroValidatorForBigDecimal implements ConstraintValidator<NegativeOrZero, BigDecimal> {

	@Override
	public boolean isValid(BigDecimal value, ConstraintValidatorContext context) {
		// null values are valid
		if ( value == null ) {
			return true;
		}

		return NumberSignHelper.signum( value ) <= 0;
	}
}
```

```java
public class NegativeOrZeroValidatorForDouble implements ConstraintValidator<NegativeOrZero, Double> {

	@Override
	public boolean isValid(Double value, ConstraintValidatorContext context) {
		// null values are valid
		if ( value == null ) {
			return true;
		}

		return NumberSignHelper.signum( value, InfinityNumberComparatorHelper.GREATER_THAN ) <= 0;
	}
}
```

**Hibernate Validator提供了对于ConstraintValidator的一个扩展：HibernateCosntraintValidator。**

这个扩展的目的是提供更多的环境信息给initialize()方法，在ConstraintValidator上只有注解本身被传了进来，HibernateCosntraintValidator增加了ConstraintDescerptor和HIbernateConstraintValidatorInitializationContext两个参数。

```java
public class PremiumAmountValidator implements HibernateConstraintValidator<PremiumAmountCheck, String> {
    private double rate;
    
    @Override
    public void initialize(PremiumAmountCheck constraintAnnotation) {
        rate = constraintAnnotation.rate();
    }

    //同时实现两个initialize，以这个为准
    @Override
    public void initialize(ConstraintDescriptor<PremiumAmountCheck> constraintDescriptor, HibernateConstraintValidatorInitializationContext initializationContext) {
        rate = constraintDescriptor.getAnnotation().rate();
        Clock clock = initializationContext.getClockProvider().getClock();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext constraintValidatorContext) {
        Boolean isValid = StringUtils.isNotBlank(value) && value.startsWith("JD");
        if(!isValid){
            constraintValidatorContext.disableDefaultConstraintViolation();
            constraintValidatorContext.buildConstraintViolationWithTemplate(
                    "保费保额错误，{rate}，{message}"
            )
                    .addConstraintViolation();
        }
        return isValid;
    }

}

```

### 约束条件组合

```java
@NotNull
@Size(min = 2, max = 14)
@CheckCase(CaseMode.UPPER)
@Target( { METHOD, FIELD, ANNOTATION_TYPE })
@Retention(RUNTIME)
@Constraint(validatedBy = {})
@Documented
public @interface ValidLicensePlate {

    String message() default "{com.mycompany.constraints.validlicenseplate}";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

}
```

```java
package com.mycompany;

public class Car {

    @ValidLicensePlate
    private String licensePlate;
    
    @NotNull
	@Size(min = 2, max = 14)
	@CheckCase(CaseMode.UPPER)
    private String licensePlate;

    //...

}
```



### 校验

1. @Validator, @Valid

```java
@PostMapping
    public ResponseEntity save(@Valid JdjrInsuredRequest dto, BindingResult bindingResult) {
        // 判断是否含有校验不匹配的参数错误
        if (bindingResult.hasErrors()) {
            // 获取所有字段参数不匹配的参数集合
            List<FieldError> fieldErrorList = bindingResult.getFieldErrors();
            Map<String, Object> result = new HashMap<>(fieldErrorList.size());
            fieldErrorList.forEach(error -> {
                // 将错误参数名称和参数错误原因存于map集合中
                result.put(error.getField(), error.getDefaultMessage());
            });
            return ResponseEntity.status(HttpStatus.BAD_REQUEST.value()).body(result);
        }
        
        return ResponseEntity.status(HttpStatus.CREATED.value()).build();
    }
    
@PostMapping
 public ResponseEntity save(@Validated({YouzanParamsValidator.class}) JdjrInsuredRequest dto, BindingResult bindingResult)
```

2. 代码中手动校验

```java
//获取validator
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();
//ConstraintViolation保存错误信息，如果所有校验都通过则Set<ConstraintViolation<T>>为空。否则默认校验所有约束并返回所有错误信息。可以修改配置为遇到第一个错误即返回
Set<ConstraintViolation<T>> violations = 	validator.validate(request,JdParamsValidator.class);
        if (!CollectionUtils.isEmpty(violations)){
            StringBuilder errorMsg = new StringBuilder();
            for(ConstraintViolation<T> constraintViolation :violations){
                errorMsg.append(constraintViolation.getMessage());
                errorMsg.append("\t");
            }
            log.error("[津投请求失败]校验失败,uuid:{},error:{}" ,uuid, errorMsg);
            result.setSuccess(false);
            result.setErrorMsg(errorMsg.toString());
        }
```

在校验完一个`ContactDetails` 的示例之后, 可以通过调用`ConstraintViolation.getConstraintDescriptor().getPayload()`来得到之前指定到错误级别了,并且可以根据这个信息来决定接下来到行为.

validator：

```java
/**
	 * Validates all constraints on {@code object}.
	 *
	 * @param object object to validate
	 * @param groups the group or list of groups targeted for validation (defaults to
	 *        {@link Default})
	 * @param <T> the type of the object to validate
	 * @return constraint violations or an empty set if none
	 * @throws IllegalArgumentException if object is {@code null}
	 *         or if {@code null} is passed to the varargs groups
	 * @throws ValidationException if a non recoverable error happens
	 *         during the validation process
	 */
	<T> Set<ConstraintViolation<T>> validate(T object, Class<?>... groups);

	/**
	 * Validates all constraints placed on the property of {@code object}
	 * named {@code propertyName}.
	 *
	 * @param object object to validate
	 * @param propertyName property to validate (i.e. field and getter constraints)
	 * @param groups the group or list of groups targeted for validation (defaults to
	 *        {@link Default})
	 * @param <T> the type of the object to validate
	 * @return constraint violations or an empty set if none
	 * @throws IllegalArgumentException if {@code object} is {@code null},
	 *         if {@code propertyName} is {@code null}, empty or not a valid object property
	 *         or if {@code null} is passed to the varargs groups
	 * @throws ValidationException if a non recoverable error happens
	 *         during the validation process
	 */
	<T> Set<ConstraintViolation<T>> validateProperty(T object,
													 String propertyName,
													 Class<?>... groups);

	/**
	 * Validates all constraints placed on the property named {@code propertyName}
	 * of the class {@code beanType} would the property value be {@code value}.
	 * <p>
	 * {@link ConstraintViolation} objects return {@code null} for
	 * {@link ConstraintViolation#getRootBean()} and
	 * {@link ConstraintViolation#getLeafBean()}.
	 *
	 * @param beanType the bean type
	 * @param propertyName property to validate
	 * @param value property value to validate
	 * @param groups the group or list of groups targeted for validation (defaults to
	 *        {@link Default}).
	 * @param <T> the type of the object to validate
	 * @return constraint violations or an empty set if none
	 * @throws IllegalArgumentException if {@code beanType} is {@code null},
	 *         if {@code propertyName} is {@code null}, empty or not a valid object property
	 *         or if {@code null} is passed to the varargs groups
	 * @throws ValidationException if a non recoverable error happens
	 *         during the validation process
	 */
	<T> Set<ConstraintViolation<T>> validateValue(Class<T> beanType,
												  String propertyName,
												  Object value,
												  Class<?>... groups);
```

**创建出来的`Validator`实例是线程安全的**

