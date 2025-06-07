#  Shell基础

​	shell是一种脚本性语言，学习shell和学习C++等语法没有其他的区别。需要注意的是shell的特性。
​	shell基础语法这里分了几个部分:

> - 变量定义
> - 运算符
> - 循环和判断
> - 函数

##  1. 变量定义

### 1.1 普通变量

​	定义变量时，变量名不需要加$符号，变量遵循其他语言的变量命名格式。(以数字命名的变量为传递变量)

```shell
# string
test1=This\ a\ string\ without\ \' or \".
echo $test1 # succes
test2='This is a string with single quotation mark.'
echo $test2 # success
test3="This is a string with \"."
echo $test3 # success
test4='This is a string with \'.'
echo $test4 # error 
```

> result:
> This a string without ' or ".
> This is a string with single\n quotation mark.
> This is a string with ".
> 在使用单引号变量时，内容不能被转义，而且不能存在单独的一个'，使用转义符号也不行，推荐使用""。

```shell
# 字符串也能获取执行命令后的结果
file="v.sh"
cmd=$(ls -a |grep $file)
echo $cmd
```

> \$ ./v.sh
> v.sh

 ### 1.2 传递变量

​	传递变量指在执行shell文件时向文件传入的变量。

```shell
./v.sh a 1
```

    这里的a、1就是传入v.sh的变量，在v.sh中获取这两个变量的值可以用
    `$1`和`$2`来获取，其中`$0`是被执行shell文件的文件名。
    `$#`可以获取传入的变量的个数

| 变量  | 含义                                  |
| ----- | ------------------------------------- |
| $0    | shell文件的文件名                     |
| \$num | 传入参数的第num个参数                 |
| \$#   | 传入文件的参数的个数                  |
| \$*   | 将所有参数组合为"arg1 ... argn"的形式 |

### 1.3 补充

​	如果要在shell中进行计算，可以用`[]`来对变量进行计算

```shell
a=1
b=2
echo $[a-b]
```

​	这里计算出的结果就是`a-b`的值（-1）。



## 2.运算符

