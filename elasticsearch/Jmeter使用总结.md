## Jmeter使用总结

#### 在Jmeter中，自动化URL编码变量

```
token=${__urlencode(${token})}
```

我们希望对${token}变量进行编码，并且只对标记变量进行编码。

#### Jmeter响应断言

jmeter在接口测试过程中，有时需要响应断言来判断接口测试得到的接口返回值是否正确。

