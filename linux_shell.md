shell脚本的第一行一般是`#!/bin/bash`，这是一个约定的标记，表明应该用bash解释器去执行这个脚本

运行方式有两种：

- 在脚本当前目录，此时就会根据脚本第一行找解释器
```
./test.sh
```

- 使用/bin/bash，此时脚本第一行就没用
```
/bin/bash test.sh
```

### 变量
定义，注意=两边不能有空格，变量可以重新被定义
```
name="tom"
```

使用，建议变量名使用花括号
```
echo ${name}
```

### 注释
使用#表示单行注释，虽然后多行注释的办法，但是不保险

### 字符串

- 单引号，原样输出，变量无效，不能出现单引号

- 双引号，可以出现变量和转义字符

### 数组

定义
```
arr=(value0 value1 value2)
或者
arr=(
value0
value1
value2
)
```

使用
```
echo ${arr[0]}
# 获取数组的所有元素
echo ${arr[@]}  或者 echo ${arr[*]}
# 获取长度
echo ${#arr[@]}
echo ${#arr[0]}
```

### test命令
test [] [[]]的区别
其中[]完全等价于test，只是写法不同。双中括号[[]]基本等价于[]，还支持正则表达式

test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

数值
|  参数   | 说明  |
|  ----  | ----  |
| -eq  | 等于则为真 |
| -ne  | 不等于则为真 |
| -gt  | 大于则为真 |
| -ge  | 大于等于则为真 |
| -lt  | 小于则为真 |
| -le  | 小于等于则为真 |

字符
|  参数   | 说明  |
|  ----  | ----  |
| =  | 等于则为真 |
| !=  | 不相等则为真 |
| -z 字符串  | 字符串的长度为零则为真 |
| -n 字符串  | 字符串的长度不为零则为真 |

文件
|  参数   | 说明  |
|  ----  | ----  |
| -e 文件名  | 如果文件存在则为真 |
| -r 文件名  | 如果文件存在且可读则为真 |
| -w 文件名  | 如果文件存在且可写则为真 |
| -x 文件名  | 如果文件存在且可执行则为真 |
| -s 文件名  | 如果文件存在且至少有一个字符则为真 |
| -d 文件名  | 如果文件存在且为目录则为真 |
| -f 文件名  | 如果文件存在且为普通文件则为真 |
| -c 文件名  | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名  | 如果文件存在且为块特殊文件则为真 |

### if 
elif else 是可选的
```
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

### 循环
```
# for
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done

# while
while condition
do
    command
done

# until
until condition
do
    command
done

# case
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2)
    command1
    command2
    ...
    commandN
    ;;
*)
    默认的处理方法
    ;;
esac

# break和continue跳出循环
```

### 函数

```
[ function ] funname [()]
{
    action;
    # return 返回，如果不加，将以最后一条命令运行结果，作为返回值
    [return int;]
}
```

函数参数
在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数
```
#!/bin/bash

funWithParam(){
    echo "方法名为 $0 !"
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    # 10开始要加花括号
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数（不包括$0的文件名） $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

