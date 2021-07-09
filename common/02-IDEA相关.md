## 1.模板配置

### 1.类模板

打开 IDEA 的 Settings，点击 Editor-->File and Code Templates，点击右边 File 选项卡下面的 Class，在其中添加图中红框内的内容：


```java
/** 
 * @author langyun 
 * @date ${YEAR}年${MONTH}月${DAY}日 ${TIME}
 */
```

### 

### 2.方法模板

不同于目前网络上互相复制粘贴的方法注释教程，本文将实现以下功能：


根据形参数目自动生成 [@param ](/param ) 注解 
根据方法是否有返回值智能生成 [@Return ](/Return ) 注解 


相较于类模板，为方法添加注释模板就较为复杂，首先在 Settings 中点击 Editor-->Live Templates。

点击最右边的 +，首先选择 2. Template Group... 来创建一个模板分组：
![](https://cdn.nlark.com/yuque/0/2021/png/1204273/1617240251143-6ea985b7-0447-4a48-a0a8-d8fdf7e4f686.png#align=left&display=inline&height=37&margin=%5Bobject%20Object%5D&originHeight=282&originWidth=1080&size=0&status=done&style=none&width=140)
在弹出的对话框中填写分组名，我这里叫做 userDefine：

![](https://cdn.nlark.com/yuque/0/2021/png/1204273/1617240276067-0c55571b-554d-462a-a08f-70647eb0461f.png#align=left&display=inline&height=25&margin=%5Bobject%20Object%5D&originHeight=161&originWidth=907&size=0&status=done&style=none&width=140)
然后选中刚刚创建的模板分组 userDefine，然后点击 +，选择 1. Live Template：

此时就会创建了一个空的模板，我们修改该模板的 Abbreviation、Description 和 Template text。需要注意的是，Abbreviation 必须为 *，最后检查下 Expand with 的值是否为 Enter 键。

![](https://cdn.nlark.com/yuque/0/2021/png/1204273/1617240296930-d8da3714-ebd0-4bb4-846c-a1fe7be2562b.png#align=left&display=inline&height=81&margin=%5Bobject%20Object%5D&originHeight=621&originWidth=1080&size=0&status=done&style=none&width=140)
上图中· Template text 内容如下，请直接复制进去，需要注意首行没有 /，且 \* 是顶格的。

```java
*
 * 
 * @author langyun
 * @date $date$ $time$$param$ $return$
 */ 
```


注意到右下角的 No applicable contexts yet 了吗，这说明此时这个模板还没有指定应用的语言：

![](https://cdn.nlark.com/yuque/0/2021/png/1204273/1617240357111-20fc3b16-5998-4911-8a6b-23ac7962b9b6.png#align=left&display=inline&height=16&margin=%5Bobject%20Object%5D&originHeight=25&originWidth=224&size=0&status=done&style=none&width=140)
点击 Define，在弹框中勾选Java，表示将该模板应用于所有的 Java 类型文件。

![](https://cdn.nlark.com/yuque/0/2021/png/1204273/1617240367492-152ab0c8-db72-4f1c-9837-97baf59f98c1.png#align=left&display=inline&height=57&margin=%5Bobject%20Object%5D&originHeight=370&originWidth=908&size=0&status=done&style=none&width=140)


还记得我们配置 Template text 时里面包含了类似于 $date$ 这样的参数，此时 IDEA 还不认识这些参数是啥玩意，下面我们对这些参数进行方法映射，让 IDEA 能够明白这些参数的含义。点击 Edit variables 按钮：

![](https://cdn.nlark.com/yuque/0/2021/png/1204273/1617240390883-f4200316-5607-471f-a6f9-baf735ef7d6d.png#align=left&display=inline&height=35&margin=%5Bobject%20Object%5D&originHeight=227&originWidth=911&size=0&status=done&style=none&width=140)
为每一个参数设置相对应的 Expression：

![](https://cdn.nlark.com/yuque/0/2021/png/1204273/1617240405570-7379901a-d0e9-4b2a-b53a-0789d9194bd4.png#align=left&display=inline&height=69&margin=%5Bobject%20Object%5D&originHeight=271&originWidth=551&size=0&status=done&style=none&width=140)
需要注意的是，date 和 time 的 Expression 使用的是 IDEA 内置的函数，直接使用下拉框选择就可以了，而 param 这个参数 IDEA 默认的实现很差，因此我们需要手动实现，代码如下：

```java
groovyScript("def result = '';def params = \"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {if(params[i] != '')result+='* @param ' + params[i] + ((i < params.size() - 1) ? '\\r\\n ' : '')}; return result == '' ? null : '\\r\\n ' + result", methodParameters())
```

```java
groovyScript("def result = '';def params = \"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {if(params[i] != '')result+='* @param ' + params[i] + ((i < params.size() - 1) ? '\\r\\n ' : '')}; return result == '' ? null : '\\r\\n ' + result", methodParameters())

```

另外 return 这个参数我也自己实现了下，代码如下：

```java
groovyScript("return \"${_1}\" == 'void' ? null : '\\r\\n * @return ' + \"${_1}\"", methodReturnType())
```

```java
groovyScript("return \"${_1}\" == 'void' ? null : '\\r\\n * @return ' + \"${_1}\"", methodReturnType())
```

注：你还注意到我并没有勾选了 Skip if defined 属性，它的意思是如果在生成注释时候如果这一项被定义了，那么鼠标光标就会直接跳过它。我并不需要这个功能，因此有被勾选该属性。
点击 OK 保存设置，大功告成！

### 3.自定义模板

## 2.插件积累