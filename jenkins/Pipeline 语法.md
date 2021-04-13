## Pipeline 语法

#### label

label 有如下两种设置方式

- 方式一

```groovy
agent{
  label 'my-defined-label'
}
```

- 方式二

```groovy
agent{
  label 'my-label1 && my-label2'
}
or
agent{
  label 'my-label1 || my-label2'
}
```

### script 标签

```groovy
script {
  Date date = new Date()
  String datePart = date.format("dd/MM/yyyy")
  echo datePart
}
```



