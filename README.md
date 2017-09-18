
正则规则文档：  
花了大量时间才整理出来的，不过多多少少有些错误和疏漏的地方，后期在整理优化


php的正则需要使用分隔符，常用的分隔符有`/`,`#`,`%`等，比如`/[a-z]+/`,  `#[a-z]+#`, `%[a-z]+%`，这三个正则表达式是等效的。如果正则里面有字符与分隔符冲突，建议可以更换不冲突的字符，这样可以不用转义。

> ### 1. 正则表达式的基本元字符
```
- [ ] { } * + . ?  ( ) | \ ^ $
```

<br/>
<br/>



> ##### 字符组`[]`

字符组`[]` ，范围表示法`-`配合字符组使用

`[]`的含义：若干个字符的集合

匹配`[]`字符组中的一个字符，可以使用范围表示法`-`

例： `[abcde]`,`[a-e]`等价，都可以匹配`abcde`等5个字符

`[字符组]`  字符组的顺序和重复都不影响匹配

`[abcdef]` 等价于 `[fedcbad]`

> 中括号的匹配：

`[` 会与最近的 `]` 匹配

`[0123]45]` 实际上分解成了 `[0123]`  `45]` ，能够匹配`145]` ,`245]` 等等，但是不能匹配`]`

例： `echo preg_match('/[0123]45]/', '345]');`  返回 1，匹配成功

<br/>
<br/>



> ##### `-` 范围表示法

范围表示法的顺序是根据ASCII码值来的，如`[a-z]`， `a`需要在`z`前面，因为ASCII表，`a`的码值是97，`z`的码值是122

例： `[0-9]`是正确的，`[9-0]`是错误的

错误的正则表达式在php中会抛出一个警告

例： 
```php
echo preg_match('/[9-0]/', '8');
# php抛出的编译失败警告： Warning: preg_match(): Compilation failed: range out of order in character class at offset 3
```
java支持`[a-z&&[^bc]]`这种写法，相当于`a-z`中排除`bc`


在字符组中部分元字符可以不转义

不用转义的元字符： `.` `*` `+` `?` `|` `()` `^` `$` `}` `[`

需要转义的元字符：`{`  `]`


不推荐使用`[0 - z]`表示数字字母，因为对于的ASCII码值中包含除数字字母以外的字符，如`[ ]` 

ASCII对照表：
http://tool.oschina.net/commons?type=4

`-` 出现的位置不同，含义不同

`-` 在最前面的时候不需要转义，例如`[a\-z]` 和 `[-az]` 是等效的，它们都能匹配`-` `a` `z` 三个字符

`[^0-9]`表示排除0-9总共10个数字，`[^-09]`表示排除`-09`三个字符，紧跟在`^`后面的`-`是不需要转义的

字符组中可以使用16进制表示法
`[\x00-\x7F]`，对应的ASCII码值为 0 - 127，7F十六进制是不区分大小写的。

<br/>

> 中文匹配的问题

网络上有一些文章使用的编码区间为`4e00 - 9fa5`，`9fa5`是比较早的标准，后来又增加了少部分字符，但是使用的话也没什么关系，因为后面的编码基本用不上,这里使用`4e00-9fff`。


例： 
```
Java             [\u4e00-\u9fff]
JavaScript       [\u4e00-\u9fff]
Python           [\u4e00-\u9fff]
PHP              [\x{4e00}-\x{9fff}]，匹配的时候需要指定unicode模式，即
在后面加u
```

utf-8 编码表：
http://www.unicode.org/charts/PDF/U4E00.pdf

<br/>

> php匹配GBK的问题

`#^[".chr(0xa1)."-".chr(0xff)."_-]+$#`
`([\xb0-\xfe][\x00-\xff])+`

GBK 亦采用双字节表示，总体编码范围为 `8140-FEFE`，首字节在 `81-FE` 之间，尾字节在 `40-FE` 之间，剔除 `xx7F` 一条线。总计 23940 个码位，共收入 21886 个汉字和图形符号，其中汉字（包括部首和构件）21003 个，图形符号 883 个。 

`GB2312-80` 是在国内计算机汉字信息技术发展初始阶段制定的，其中包含了大部分常用的一、二级汉字，和 9 区的符号。该字符集是几乎所有的中文系统和国际化的软件都支持的中文字符集，这也是最基本的中文字符集。其编码范围是高位`0xa1－0xfe`，低位也是 `0xa1-0xfe`；汉字从 `0xb0a1` 开始，结束于 `0xf7fe` 

GBK 编码表：
http://www.qqxiuzi.cn/zh/hanzi-gbk-bianma.php

参考资料：
http://ir.hit.edu.cn/~taozi/bianma.htm

<br/>

> 字符组的简记法

对于ASCII编码：

`\d(digit)`  等价于`[0-9]`

`\w(word)`  等价于`[0-9a-zA-Z_]`，`\w`能匹配下划线

`\s(space)`  基本等价于`[ \t\n\r\v\f]`，其中包含空格

`\d\w\s` 和`\D\W\S`互补

对于unicode编码：

`\d`可以匹配 `[0-9]`，也可以匹配全角的1 2 3 4
等价于：`[\p{Nd}]`

`\w`可以匹配`[0-9a-zA-Z_]`，也可以匹配中文字符
等价于： `[\p{Ll}\p{Lu}\p{Lt}\p{Lo}\p{Lm}\p{Nd}\p{Pc}]`

`\s` 可以匹配`[ \t\n\r\v\f]`，也可以匹配全角的空白符号
等价于：`[ \t\n\r\v\f\x85\p{Z}]`

`\p{L}` 代表任意语言中的字母字符(包括英文字母和汉字)

`\p{Nd}` 代表0-9的数字，包括全角数字

`\p{Pc}` 代表类似下划线之类的标点符号

`\p{Z}`  代表分隔符（比如空格、换行等）


参考资料：
http://www.regular-expressions.info/unicode.html

<br/>
<br/>



> ##### 量词`{} * + ? .`

`{m,n}`，最短m个字符，最长n个字符，大括号内不能有空格，有空格则无法匹配，n 必须大于 m，否则会报错

写法如：`{m,n}` `{m,}`    `{m}`，部分语言可能支持`{,n}`，但是不推荐，推荐用`{0,n}`，php不支持`{,n}`这种写法

例：
```php
echo preg_match("/^ab{2,4}/", "abb");
# 返回1

echo preg_match("/^ab{2,4}/", "abbbb");
# 返回1

echo preg_match("/^ab{2}/", "abbbb");
# 返回1

echo preg_match("/^ab{2,}/", "abbbb");
# 返回1

preg_match_all("/^ab{2}/", "abbbb", $match);
# 返回 Array ( [0] => Array ( [0] => abb ))

preg_match_all("/^ab{2,}/", "abbbb", $match);
# 返回 Array ( [0] => Array ( [0] => abbbb ) )
```

`*` 表示可能出现，也可能不出现，出现次数没有上限，等价于`{0,}`

例：
```php
echo preg_match("/^ab*/", "abbbbb");
# 返回1，匹配成功
# b出现的次数为0次或以上都能够匹配
```

`+` 表示至少出现一次，出现次数没有上限，等价于`{1,}`

例：
```php
echo preg_match("/^ab+/", "ab");
# 返回1，匹配成功
# b出现的次数为1次或以上都能匹配
```

`?` 表示至多出现一次，也可能不出现，等价于`{0,1}`

例：
```php
echo preg_match("/^ab?/", "a");
# 返回1，匹配成功
# b出现的次数为0次或者一次都能匹配
```


`.` 表示匹配任意字符，除`\n`外，匹配任意字符`[\s\S]`  `[\d\D]`  `[\w\W]`

匹配任意字符是相对应单字节的ASCII编码的，实际`.`匹配的是除`\n`外的任意字节
例：
```php
echo preg_match("/^.$/", '测');
# 返回0，匹配失败 
echo preg_match("/^.$/u", "测");
# 返回1，匹配成功
echo preg_match("/^.*$/", "测");
# 返回1，匹配成功
echo preg_match("/^.{3}$/", "测");
# 返回1，匹配成功
```
java不需要显示指定unicode模式，所有可以直接用`.`匹配'测'字

<br/>

> 匹配优先量词，也叫贪婪匹配

贪婪匹配是在匹配成功的情况下仍然尽可能多的匹配，在匹配的时候会保存更多的状态已便在无法匹配的时候回溯。  
例：
```php
$str = '<script>
alert("1");
</script>
test
<script>
alert("2");
</script>';
$str = preg_replace("#<script>[\s\S]*</script>#", '', $str);
# 该示例会匹配出非预期的结果。会匹配开始<script>到结束</script>里面的所有内容，最后返回一个空结果
```

<br/>

> 忽略优先量词，也叫非贪婪匹配

非贪婪匹配在匹配成功的情况下尽可能少的匹配
```php
$str = preg_replace("#<script>[\s\S]*?</script>#", '', $str);
```


忽略优先量词写法  
`*?`    `+?`    `??`    `{m,n}?`

<br/>

> 占有量词

优先占有，占有后无法回溯，使用形式 `?+`、`*+`、`++`和`{m,n}+`  
例：
```php
$reg = "#(\.\d\d[1-9]?+)\d+#";
$str = '0.123';
preg_match_all($reg, $str, $match);
print_r($match);
# 结果：无法匹配
```

参考资料：
http://blog.csdn.net/shangboerds/article/details/7603022


括号`()` ，多选 `|`，一般配合括号使用，也可单独使用  
多选结构，`(a|b|c)`， 可以匹配 `a` 或者 `b` 或者 `c`   
`^(ab|cd)` 不等于 `^ab|cd`，`^ab|cd` 等于 `(^ab|cd)`  

<br/>

> 引用分组

使用括号后，正则表达式会保存每个分组匹配的内容  
括号的分组根据从左到右的顺序，每对括号一个分组  

<br/>

> 固化分组

固化分组内的内容不能回溯，使用形式：`(?>…)`  
例：  
```php
$reg = "#(\.\d\d(?>[1-9]?))\d+#";
$str = '0.123';
preg_match_all($reg, $str, $match);
print_r($match);
# 结果：无法匹配
```

<br/>

> 反向引用分组

允许在正则表达式内部引用之前的捕获分组匹配的文本，其形式是`\num` 或者 `$num`
```
php        表达式中为 \num， 替换中为 \num 或者 $num
java       表达式中为 \num， 替换中为 $num
javascript 表达式中为 $num， 替换中为 $num
php 在替换中可以使用 \${num} 消除二义性
```

例如:   
`^([a-z])\1$`可以匹配`aa`  
`\10`以上在不同语言可能存在二义性  

替换中使用反向分组  
```php
$test = '/s_test/s_3trtfgftkjiujmd.jpg';
$test = preg_replace("#(/s_)([a-zA-Z0-9]+\.)#", "/m_\${2}", $test);
```

表达式中使用反向分组  
```php
$test = "<html>test11</html>";
preg_match_all("#<(html)>.*</\\1>#", $test, $match);
print_r($match);
# 返回：
# Array( [0]=>Array( [0]=><html>test11</html>) [1]=>Array([0]=>html))
```

<br/>

> 命名分组

java7之前版本不支持命名分组  
javascript不支持命名分组  
php分组的记法 `(?P<name>...)`，分组的引用`(?P=name)`, 5.2.2以后支持 `\k<name>` 或者 `\k'name'`，5.2.4以后可以使用`\k{name}`或者 `\g{name}`。  
php只支持表达式中使用命名分组  
示例：
```
$test = "<html>test11</html>";
preg_match_all("#<(?P<hl>html)>.*</(?P=hl)>#", $test, $match);
preg_match_all("#<(?P<hl>html)>.*</\k{hl}>#", $test, $match);
preg_match_all("#<(?P<hl>html)>.*</\k'hl'>#", $test, $match);
preg_match_all("#<(?P<hl>html)>.*</\k<hl>>#", $test, $match);
preg_match_all("#<(?P<hl>html)>.*</\g{hl}>#", $test, $match);
```

<br/>

> 非捕获分组

正则表达式会把()内的所有内容都保存起来，作为后续的引用，如果不需要引用，则可以使用非捕获分组，则匹配的内容不会保存起来，提高效率。非捕获分组的形式 `(?:...)`  
例：
```php
$test = '/s_test/s_3trtfgftkjiujmd.jpg';
preg_match_all("#(?:/s_)([a-zA-Z0-9]+\.)#", $test, $match);
print_r($match);
```

<br/>
<br/>

> ##### 其他元字符

开头 `^`  
匹配字符串起始位置  
结尾 $  
匹配字符串结束位置  
`\`  
转义符号  

<br/>
<br/>
<br/>



> ### 2. 元字符的转义

需要转义的元字符 `-` `[` `]` `^` `$` `(` `)` `+` `.` `?` `|` `\` `{` `}` `*`

`{m,n}`的转义形式为 `\{m,n}`

`[mn]`的转义形式为 `\[mn]`

`(a|b)`的转义形式为`\(a\|b\)`，需要注意括号的转义形式与其他成对出现的不同
其他字符的转义形式为`\-`  `\^`  `\$`  `\+`  `\.`  `\?`  `\|`  `\\`  `\*`

在正则表达式中，双引号中的反斜杠(`\`)需要进行双重转义，正则表达式需要进行转义，字符串中的特殊字符(`\` `"` `'`)需要进行转义。部分语言提供了使用原生字符串，即不需要进行字符串层面的转义，如python的r模式。`re.search(r"^[0\-9]$", '-')`;

先进行字符串层面的转义，转义完成后的字符串再进行正则转义

字符串中的转义(`\` `\t` `\n` `'` `"`)

`\` 必须转义， `\t` `\n` 可以不转义， `\b`必须转义成 `\\b` 才能作为单词边界，否则作为退格符

示例：
```php
echo preg_match("#\\\\n#", "\\n");
# 返回1，匹配成功，匹配的是字符串 '\n'
echo preg_match("#\\n#", "\\n");
# 返回0，匹配失败
echo preg_match("#\\n#", "\n");
# 返回1，匹配成功，匹配的是换行符 \n 
echo preg_match("#\n#", "\n");
# 返回1，匹配成功
```

单双引号有区别：
```php
echo preg_match('#\\\n#', '\nabc');
# 返回1, 匹配成功
echo preg_match('#\\\n#', "\nabc");
# 返回0，匹配失败
echo preg_match('#\\\\n#', '\nabc');
# 返回1，匹配成功
echo preg_match('#\\\\n#', "\nabc");
# 返回0，匹配失败

echo preg_match("#\\\n#", '\nabc');
# 返回0，匹配失败
echo preg_match("#\\\n#", "\nabc");
# 返回1，匹配成功
echo preg_match("#\\\\n#", '\nabc');
# 返回1，匹配成功
echo preg_match("#\\\\n#", "\nabc");
# 返回0，匹配失败

# 在php 源码 ext->pcre->pcrelib->pcre_internal.h 文件中有这么一段注释：
/* In ASCII/Unicode, linefeed is '\n' and we equate this to NL for
compatibility. NEL is the Unicode newline character; make sure it is
a positive value. */
# 为了确保兼容性，我们会把 \n 作为 NL来处理
```

unicode换行符指南：
http://unicode.org/standard/reports/tr13/tr13-5.html

<br/>
<br/>
<br/>


> ### 3. 断言

常见的三类断言：单词边界，行起始/结束位置，环视

`[由于markdown语法的问题，$ 原样显示]`

单词边界, 表示符号：`\b`  
非单词边界，表示符号：`\B`，使用频率较少  
例：  
`\btest\b` ，只能匹配单词`test`
  

行起始/结束位置，表示符号：`^` $

`^` 匹配整个字符串的起始位置  
$   匹配整个字符串的结束位置  
如果是多行模式，则`^`可以匹配整个字符串的起始位置，也可以匹配换行符之后的位置。  
`\A`的功能和`^`相同，但是不受多行模式的影响。即使遇见换行也匹配字符串的起始位置  
`\Z`等价于非多行模式下的$  
`\z`不管是否是多行模式，只匹配整个字符串的结束位置  
javascript只支持`^` $  

<br/>
<br/>



> ##### 环视

环视类似于单词边界，本身不匹配任何字符，环视结构的`( )`不会被分组捕获。但是如果环视结构里面还有`( )`，则该`( )`是可以被捕获的。如： `<(?=(img|br))[^>]*/>`

多个环视可以并列，可以包含。

<br/>
<br/>

> ##### 四种环视

> 肯定顺序环视：`(?=...)` 向右判断  

向右匹配，当前位置的右侧需要存在能匹配的文本

例：
```php
echo preg_match('#<(?=/)([^>]*)>#','<img />');
# 返回0，匹配失败
echo preg_match('#<(?=/)([^>]*)>#','</img >');
# 返回1，匹配成功
echo preg_match('#<(?=/br)([^>]*)>#','</br>');
# 返回1，匹配成功
```

<br/>

> 否定顺序环视：`(?!...)` 向右判断

向右匹配，当前位置的右侧不能存在能匹配的文本

例：
```php
echo preg_match('#<(?!/)([^>]*)>#','<img />');
# 返回1，匹配成功
echo preg_match('#<(?!/)([^>]*)>#','</img >');
# 返回0，匹配失败
echo preg_match('#<(?!/br)([^>]*)>#','</br>');
# 返回0，匹配失败
# <后面一个字符不能是/
```

<br/>

> 肯定逆序环视：`(?<=...)` 向左判断，`(?<=...)` 表达式前面无内容

向左匹配，当前位置的左侧需要存在能匹配的文本

例：
```php
echo preg_match('#(?<=br/>)([^>]*)>#','<br/><img/>');
# 返回1，匹配成功
```

<br/>

> 否定逆序环视：`(?<!...)` 向左判断，`(?<!...)` 表达式前面无内容

向左匹配，当前位置的左侧不能存在能匹配的文本

例：
```php
echo preg_match('#(?<!br/>)<img([^>]*)>#','<br><img src="..."/>');
# 返回1，匹配成功
echo preg_match('#(?<!br/>)<img([^>]*)>#','<br/><img src="..."/>');
# 返回0，匹配失败
```

注：nginx只支持否定逆序环视，左侧可以存在文本  
例：  
```
location ~ .+/resources/(.*) {
        rewrite  (.+)(?<!View)/resources/(.*)$  $1/View/resources/$2  last;
}
```

<br/>

> 支持情况

常用的语言大都支持顺序环视，javascript不支持逆序环视。尽量避免在逆序环视中使用过于复杂的表达式，各种语言对逆序环视里面表达式的支持情况不太一样，有可能会不支持。

<br/>
<br/>
<br/>



> #### 4. 匹配模式

常用的正则表达式匹配模式：不区分大小写模式，单行模式，多行模式，注释模式

> s单行模式

java和python中称为DOTALL(点号通配)，比较少用  
javascript不支持  
特点：单行模式中, 点号(`.`)可以匹配包括 换行符(`\n`) 在内的所有字符  
例：  
php写法：
```
#<script>.*</script>#s，如果分隔符是 / 则需要转义，否则会报错
```
java写法：  
```
(?s)<script>.*</script>
```
或   
```java
matcher = Pattern.compile("<script>.*</script>", Pattern.DOTALL).matcher(....);
# 匹配：
<script>test
test
</script>
```

<br/>

> i不区分大小写模式

特点：匹配的时候不区分大小写  
例：  
```
echo preg_match("/Test/i", 'test');
# 返回1
```
`(?i)`可以放在正则中间，可使后面的所有匹配都不区分大小写，php不支持  
例：  
`This is a(?i) Test`，`a`之后可以都不区分大小写  
php和javascript的写法：  
`/reg/i`  
java的写法：  
`(?i)reg`  
或  
```
matcher = Pattern.compile("Test", Pattern.CASE_INSENSITIVE).matcher(....);
```

<br/>

> m多行模式

特点：多行模式下^ $可以匹配字符串内部某一行文本的起始位置和结束位置。  
例：  
```
$str = "<script>test
test
test1234</script>";
```
php写法：  
```php
preg_match_all('#^(?:test).*</script>$#m', $str, $match);
# 匹配的结果为 test1234</script>
```
java的写法：
```
(?m)^(?:test).*</script>$
```
或
```
matcher = Pattern.compile("<script>.*</script>", Pattern.MULTILINE).matcher(....);
```
javascript的写法：
```
/^(?:test).*<\/script>$/m
```

<br/>


> x注释模式

特点：可以给过于复杂的正则表达式添加注释，表达式`(?#comment)`  
例：
```php
$reg = "%
^    #start
\d+    #digit
$    #end
%x";
// $reg = "%^(?#start)\d+(?#digit)$(?#end)%";
echo preg_match($reg, '12377456');
```

java的写法：
```
(?x)reg
```
或
使用`Pattern.COMMENTS`

javascript的写法：
```
/reg/x
```

<br/>

> u unicode模式

特点：用于匹配unicode编码，php中有该模式  
例：  
```
echo preg_match("#[\x{4e00}-\x{9fff}]#u", "测");
# 返回1，匹配成功
```

<br/>
<br/>
<br/>



> #### 5. 匹配原理

有穷自动机  
参考资料：http://metc.gdut.edu.cn/compile/cmpl3/3-3.htm


分为两种，NFA(非确定型有穷自动机)和DFA(确定型有穷自动机)  
区别：  
NFA，在状态转移的过程中，状态可以是不确定的。  
DFA，在状态转移的过程中，状态是确定的。  
使用NFA，需要保存所有的状态，因为状态是不确定的，确保错误的时候可以回溯。  
使用DFA，任何时候状态都是确定的，所有不存在回溯。捕获，反向引用，环视，忽略优先量词等功能都是不支持的。  

MySQL只支持DFA，各个语言的正则引擎都支持NFA  

理解回溯：
```
[.(9)|^$*+?}[\]](ab|cd)
(.|)^${}*+.[]?cd
```

<br/>
<br/>
<br/>

> #### 6. 效率

使用非捕获分组

减少回溯的次数，如使用忽略优先量词，尽量避免写 `.*`匹配，因为会造成回溯，过多的回溯次数会降低性能

避免过于复杂的正则，能排除的字符最好排除

<br/>


> 附：  

简单的正则匹配程序
https://github.com/caglar/simple_regex

php源码正则实现
https://github.com/php/php-src.git
目录： ext -> pcre

工具: http://www.regexbuddy.com/

相关书籍：
正则指引 
http://item.jd.com/10972570.html

精通正则表达式
http://item.jd.com/11070361.html


参考资料：
http://pubs.opengroup.org/onlinepubs/009695399/basedefs/xbd_chap09.html

http://www.infoq.com/cn/news/2011/07/regular-expressions-6-POSIX

http://www.regular-expressions.info/

https://en.wikipedia.org/wiki/Regular_expression








