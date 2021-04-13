## 注解使用

#### 注解模板

```java
@Target(ElementType.FIELD)
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Name {
	String name() default "";
}
```

#### 获取注解标记的元素

```
private Map<String,Field> getFields(Object configBean) {
	Class<?> clazz = configBean.getClass();
	Field[] declaredFields = clazz.getDeclaredFields();
	Map<String,Field> fields = new HashMap<>();
	for(Field field : declaredFields) {
		Name annotationName = field.getAnnotation(Name.class);
	}
}
```

