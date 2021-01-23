---
layout: post
title: Shell基本语法
tags: Shell  

---
<!-- TOC -->

- [一、变量](#一变量)
    - [字符串](#字符串)
    - [数组](#数组)
- [二、参数传递](#二参数传递)
- [三、运算](#三运算)
- [四、逻辑运算符](#四逻辑运算符)
- [五、条件分支](#五条件分支)
- [六、循环控制](#六循环控制)
- [七、函数](#七函数)
- [八、Shell 输入/输出重定向](#八shell-输入输出重定向)
- [九、Shell 文件包含](#九shell-文件包含)

<!-- /TOC -->
# 一、变量

```
1. 变量命名之间不能有空格
2. 使用变量需要使用$符号，最好使用{}来规定变量的边界，可以在字符串中直接使用变量
3. 只读变量用 readonly  修饰
4. 删除变量 unset （unset 命令不能删除只读变量）
5. 字符串推荐使用双引号（可以有变量，也可以有转义符）
```



## 字符串

```
拼接字符串
your_name="Lenjor"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

获取字符串长度
echo ${#string} #输出 4

提取子字符串
string="LenjorLiang is a great programer"
echo ${string:1:5} # 输出 enjor

查找子字符串
a="The cat sat on the mat"
test="cat"
awk -v a="$a" -v b="$test" 'BEGIN{print index(a,b)}'
```



## 数组

```
array_name=(value0 value1 value2 value3)
valuen=${array_name[n]}
使用 @ 或 * 符号可以获取数组中的所有元素，例如：
echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"

# 指定下标访问
my_array[0]=A
```



# 二、参数传递

```
格式: $n 	n为数字，0默认是文件名称，第一个参数从1开始
$1  标识第一个参数
$$  执行进程号
$*	获取全部参数
$#	参数个数
```



# 三、运算

```
val=`expr 2 \* 12 - 2`
echo $val   # 22

1. 使用 expr 来计算
2. 乘法需要有反斜杠转义
3. 每个算子之间必需要有空格分隔

```

# 四、逻辑运算符

```
与	-a
或	-o
非	！
```

# 五、条件分支

```
if [condition]
	then 
		[...]
	elif [condition]
	then
		[...]
	else
  	[...]
fi;  	
```

# 六、循环控制

```
# for条件循环
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done


# while条件循环
while condition
do
    command
done

# 直到condition为true停止
until condition
do
    command
done


# case使用
case 值 in
模式1）
    command
    ...
    commandN
    ;
模式2）
    command
    ...
    commandN
    ;;
esac
```



# 七、函数

```
函数定义如下：
[ function ] funname [()]
{
    action;
    [return int;]
}

# 一个简单的函数例子
function myfunc(){
	name=$1;
	echo "Hello $name";
}
# 函数的调用，在后面可以添加参数，在函数体里面使用 $n 接收
myfunc Lenjor;


function myfunc(){
	echo "函数返回值：$1"
	return $1;
}
val0=0;
if (myfunc $val0)
	then echo "true";
else echo "false";
fi;

val=1;
if (myfunc $val)
	then echo "true";
else echo "false";
fi;

输出结果：
函数返回值：0
true
# 函数的返回值只能是0~255的int值，如果没有返回，会返回最后一行执行的结果，注意函数返回值作为condition的时候，只有0的时候才是true，其他的值都是false
```



# 八、Shell 输入/输出重定向

重定向命令列表如下：

| 命令            | 说明                                               |
| :-------------- | :------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |



# 九、Shell 文件包含

```
. filename   # 注意点号(.)和文件名中间有一空格
或 source filename

在Linux里面，经常会使用 source ~/.bash_profile 来重新加载配置
```

