## BitBake recipes语法

完整BitBake 语法描述，请参考用户手册的 [语法和操作符号](https://www.yoctoproject.org/docs/2.6.2/bitbake-user-manual/bitbake-user-manual.html#bitbake-user-manual-metadata)

### 变量赋值和操作符

变量赋值准许将值赋给变量。赋值可以是静态文本，也可以包含其他变量的内容。除了赋值，还支持追加和前增操作符。

```bash
S = "${WORKDIR}/postfix-${PV}"
CFLAGS += "-DNO_ASM"
SRC_URI_append = " file://fixup.patch"
```

### 函数

函数提供了一系列要执行的操作。通常使用函数来覆盖任务函数的默认实现或补充默认函数(即追加或前增到现有函数)。标准函数使用sh shell语法，但也可以访问OpenEmbeded(meta-oe)变量和内部方法。

以下是sed 食谱中的示例函数：

```bash
do_install() {
	autotools_do_install
	install -d ${D}${base_bindir}
	mv ${D}${bindir}/sed ${D}${base_bindir}/sed
	rmdir ${D}${bindir}/
}
```

### 关键字

BitBake 食谱仅使用几个关键字。你可以使用关键字包含常用函数(inherit),从其他文件加载食谱的一部分(include和require)以及将变量导出到环境(export)。

```bash
export POSTCONF = "${STAGING_BINDIR}/postconf"
inherit autoconf
require otherfile.inc
```

### 行继续(\\)

使用反斜杠(\\)字符将语句拆分为多行。将斜杠字符放在要在下一行继续的行的末尾。

```bash
#注意 斜杠字符后面不能包含任何字符，包括空格或制表符。
VAR = "A really long \
	line"
```

### 使用变量(${VARNAME})

使用`${VARNAME}`语法访问变量的内容：

```bash
#注意 重要的是要理解以这种形式表示的变量的值不会立即扩展替换。这些表达式的扩展会在稍后按需发生
#（例如，通常在引用变量的函数执行时）。此行为可确保变量值最适合最终使用它们的上下文。在极
# 少数情况下，你确实需要立即扩展变量表达式，您可以使用:=运算符而不是=在进行赋值时，但通常不需要这样做。
SRC_URI = "${SOURCEFORGE_MIRROR}/libpng/zlib-${PV}.tar.gz"
```

### 引用所有赋值("value")

在所有变量使用双引号(例如:"value")赋值。以下是一个例子：

```
VAR1 = "${OTHERVAR}"
VAR2 = "The version is ${PV}"
```

### 条件赋值(?=)

条件赋值用于为变量赋值，但仅限于当前未设置变量的情况。使用问号后跟等号(?=) 来进行条件赋值的"软"赋值。通常，"软"赋值在local.conf文件中用于从外部环境获取的变量。

如下：VAR1 如果当前为空，则设置为"New value".但是，如果VAR1已经设置，它将保持不变：

```bash
VAR1 ?= "New value"
```

在下一个示例中，`VAR1`保留值“`Original value`”：

```bash
VAR1 = "Original value"
VAR1 ?= "New value"
```

### 追加(+=)

使用加号后跟··等号(+=)将值附加到现有变量。

```bash
# 注意 此运算符在变量的现有内容和新内容追添加空格。
SRC_URI += "file://fix-makefile.patch"
```

### 前增(=+)

使用等号后跟加号(=+)将值前增到现有变量。

```bash
#注意 此运算符在新内容和变量的现有内容之间添加空格。
VAR =+ "Starts"
```

### 追加(_append)

使用_append运算符将值附加到现有变量。此运算符不会添加任何额外空格。此外，此运算符在所有的+= 和 =+ 运算符起效后起效，并且在所有 = 号赋值发生之后才起效。

以下示例是显式地在开头添加空格，以确保附加值未与现有值合并

```bash
SRC_URI_append = " file://fix-makefile.patch"
```

您还可以将`_append`操作符与覆盖一起使用，这将导致仅对指定的目标或机器执行操作：

```bash
SRC_URI_append_sh4 = " file://fix-makefile.patch"
```

### 前增（`_prepend`）

使用_prepend运算符将值前增到现有变量。此运算符不会添加任何额外空格。此外，此运算符在所有的`+=`和`=+`运算符起效后起效，并且在所有`=`号赋值发生之后才起效。

以下示例是显式地在末尾添加空格，以确保前置值未与现有值合并：



### 输出日志

```python
bb.plain("Sstate summary: Wanted %d Found")
```





























