```
#!/bin/bash
select var in red green blue $'exit'	# $'exit'防止歧义
do
	# echo $REPLY:$var
	case $REPLY in
		1)
			echo "红"
			;;
		[2-3])
			echo "绿"
			;;
		4)
			break
			;;
		*)
			echo "不知道"
			;;
	esac
done
```
```
#	function 函数名
#	{
#		语句
#		return 值
#	}

#	函数名 参数1 参数2
#	返回值是 $? （0-255）之间，一般只用于执行成功或失败

function add	# 或add()，支持管道 。。。| function add
{
	rst = $(($1+$2))	# “局部变量”全局可用
	return $((20+30))
}>a.txt	#支持重定向
```
```
#	调用
add 111 222	# 支持管道。。。| add 111 222
echo $rst	# 333
echo $?		# 50
```
```
declare -f add	# 返回函数的实现
declare -F add	# 返回函数名（若存在）
```
```
#################################
#	% ulimit -a
#	i18n
#	unset 删除变量
#
```
```
find 目录 -name 文件名字
			-type 文件类型
			-perm 文件权限
			-ctime +n/-n	# 修改时间，单位小时，+代表大于，-代表小于
# 备份24小时内的文件			
find . -ctime -24 | while read file
do
	zip tt.zip $file
done
```
```
gcc	编译器(CL.exe)
ld	连接器(link.exe)
```
```
# -z -n 判断字符串空与非空
# 命令替换符 $(ls) `ls` 转换成数组 ( `ls` )
# 文件测试 -ot(old than) -nt(new than)
```
