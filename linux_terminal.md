# Linux Terminal
- 请体会两种括号<>和[]的含义，尖括号代表一个“对象”，方括号代表“可选的对象”
## 一、	Linux命令基础：
- 在Linux中，每一个(外部)命令都是一个可执行文件（程序）
- 内部命令没有可执行文件
- - -
- 用；连接多个命令
- 用引号标记一个整体（保持原样，单引号更严格，在跟环境变量有关时使用）
-	‘%’为命令提示符 prompt，每一种shell都有自己的提示符，且可以配置
```
用反引号`<CMD>` 表示执行<CMD>，并取输出结果
% pwd
/root
% a=`pwd`
% echo ${a}
/root
```
- 按光标上下键翻看历史命令，按回车重新执行！
```
% history	显示历史命令，它存在用户主目录下的.bash_history文件中
% !<NUM>	重新执行第<NUM>个历史命令
```
- 在命令后面加&，表示让此命令在后台运行
- Ctrl+z	挂起当前任务
```
% jobs	显示后台任务，包括挂起任务
```
- 	[ ]：后台任务的编号
- 	Done：执行完成
- 	Running：运行与就绪
- 	Stopped：挂起
- 	+-表示优先级，什么都么有的优先级最低
```
% fg	把带+的任务调度到前台执行
% fg 6	把6号任务调度到前台执行
% bg	把挂起的任务在后台运行
```
- 命令后面可以带参数和选项，也可不带
	- 参数一般用来指定目标，选项一般用来做细微调整
	- 选项用‘-’打头，参数不用
		e.g.
    ```
		% ps -f -p <pid>
			-p 和 -f 是选项,可以连写成-pf或-fp
			pid（进程ID）是参数
		% ls /mnt /usr
			带两个参数
    ```
- Ctrl+c		终止当前命令
```
% kill PID	向某个进程（PID）发送信号SIGTERM终止进程
% kill %1	终止1号后台任务
```
```
% man	打开手册（帮助文档） manual
		e.g.
		% man date
	按h显示 manual page 的帮助文档
	按q退出
```
```
% ps	查看进程执行状态，即当前命令的执行状况 present status
```
- F
- S：状态。S-睡眠；R-运行；T-挂起；Z-僵尸进程（已经结束但还占用资源未回收）
- UID：
- PID：进程ID
- PPID：父进程ID
- C：
- PRI：
- NI：
- ADDR：
- SZ：
- WCHAN：
- TTY：终端编号
- TIME：进程运行时间长度
- - -
shell本身也是命令，输入shell的名称来运行之，e.g.
```
% bash
% ksh
% csh
% sh
% tcsh
% ps
```
可连续执行多个shell，不过只有最后一个运行的shell起作用
- - -
```
% ps -e -l	表示列出所有终端的进程状态，用长格式
% ps -f		显示完整的命令（带参数和选项），STIME开始时间
% ps -p PID	根据进程ID查看
```
```
% gnome-terminal 命令行打开终端
	init是初始化进程，PID=1 。UNIX里有sched（调度进程），PID=0，但Linux里没有
% exit	推出当前shell，退出最后一个shell会关闭终端
	每个系统都有默认的shell，root（管理员）可以为每个用户指定“登录shell”
% exec ksh	切换至ksh，而非“覆盖”（执行% ps 查看不同效果）
```
```
% clear	清理（清空）屏幕，或按ctrl+l
```
```
% date	显示日期
% cal	显示日历
% cal YEAR	查看整年的日历
% cal MONTH YEAR	参看某月的日历
	e.g.% date +%D
		% date +%F%A%T 或 % date +“%F %A %T”
```
```
% sleep SLEEP_TIME(seconds)
```
## 二、	Linux账户操作：
root为linux的系统管理员，俗称“根用户”
```
% passwd	修改密码
```
- 密码存在 /etc/passwd
- USERNAME:PWD(X):UID:GID:注释:主目录:登录shell
- 密码存在shadow文件里面
- 组信息保存在 /etc/group
- - -
```
% id	查看当前用户信息
% id root	查看根用户信息
```
```
% su -root	切换至根用户
```
```
% who	显示当前在线用户的详细信息（正在使用这个主机）
% who am i	显示自己的详细信息
% whoami	显示自己的简略信息（仅用户名）
% users		显示所有在线的用户名
% w		显示所有在线用户更加详细的信息 who where when what(are they doing)
```
- Linux是多用户操作系统，多个用户可以远程登录！
- 同一个用户可多次登录，但终端编号不同！

## 三、	Linux文件操作：
- Linux中的“目录”即Windows中的“文件夹”
- 注意：目录名和文件名严格区分大小写！
- 在Windows中，把外存分虚拟成不同的磁盘，但Linux中只有“根目录”，用斜杠‘/’表示
- /dev/disk 目录中可以看到各个磁盘
- 一个点：./ 或 . 或什么都不加，表示当前目录，即最后的‘/’可要可不要
- 两个点：../ 或 .. 表示上一级目录，../../ 表示上一级目录的上一级目录
- 波浪号：~ 表示主目录Home，由管理员指定
- 路径：
	- 绝对路径：/目录名/子目录名/。。。/	如上述的：/dev/disk/
	  - 除了第一个‘/’，之后的‘/’是分割符，注意Windows里面的分隔符是反斜杠‘\’
	- 相对路径：./目录名/子目录名/。。。/ 或 目录名/子目录名/。。。/
	- 用路径+文件名唯一标识一个文件
```
% pwd	打印当前工作目录 present working directory
% cd	改变当前工作目录 change directory
	e.g.% cd 或 %cd ~	表示回到主目录
		% cd ~uid	表示进入uid这个用户的主目录
```
```
% ls	列出当前目录下的文件和子目录
% ls -l	长格式
% ls -F	给文件或目录打标记
	/ 目录；* 可执行文件；| 管道；@ 软链接（符号链接，快捷方式）；什么都没是普通文件
% ls -a	列出所有的文件盒子目录，包括影藏文件，由点(.)打头
% ls -A	跟小写a一样，但不包括.和..
% ls -r	按字母降序排列
% ls -t	按时间顺序排列
% ls -R	列出指定目录开始的目录树
% ls -d DIR_NAME	查看目录本身而非进入目录查看

总计 磁盘块数（512B/块）
-rwxrwxrwx 硬链接数 用户名 组名 大小 创建时间 名字
-：文件		r：读
d：目录		w：写
l：软链接	x：可执行
p：管道文件
```
```
% chmod [ugo][+-=][rwx]{1,3} FILE_NAME	change mode
% chmod o=- FILE_NAME		剥夺外组所有权限
% chmod u-w,o+r,g+w FILE_NAME	批处理用逗号隔开
% chmod ugo+x FILE_NAME		所有人授予执行权限
% chmod a+x FILE_NAME		所有人授予执行权限
	u 自己；g 同组；o 外组（其他）
% chmod 0700 FILE_NAME		最好0打头
```
```
% mkdir DIR_NAME_1 DIR_NAME_2 DIR_NAME_2 ...
% mkdir DIR_NAME/DIR_NAME	第一个DIR_NAME必须存在
% mkdir -p DIR_NAME/DIR_NAME	第一个DIR_NAME可以不存在
```
```
% touch FILE_NAME_1 FILE_NAME_2 FILE_NAME_3 ...	创建空文件

% date > FILE_NAME	> 表示重定向。把左边命令(date)的输出写到右边文件中。每次都会清空原有内容在写入新内容。
% CMD >> FILE_NAME	>> 表示追加
% echo hello > FILE_NAME	回写（任意输出）
% echo "hello world" > FILE_NAME;cat FILE_NAME	分号隔开多条命令,用双引号保留空白字符。

% date > /dev/tty 	/dev/tty 这个文件表示终端
			/dev/null 空设备文件（黑洞文件）
```
```
% cat FILE_NAME		查看文件内容，直接显示至文件末尾
% more FILE_NAME	分屏显示：满一屛暂停，空格键往下走一屛，回车键走一行，显示到文件末尾或按q退出
% less FILE_NAME	比more多些功能：用方向键上下翻页。more不能向前翻页
% head -5 FILE_NAME	查看文件的前5行，不带选项，默认前10行
% tail FILE_NAME	查看文件后几行，默认后10行
```
```
% wc FILE_NAME		统计行数、单词数（空白符分隔）和字符数 word count
% wc -l FILE_NAME	只要行数
% wc -w FILE_NAME	只要单词数
% wc -c	FILE_NAME	只要字符数
```
```
% grep STR FILE_NAME	在FILE_NAME中查找包含STR的行
% grep -i STR FILE_NAME	-i 表示忽略(ignore)大小写
% grep -v STR FILE_NAME	不包含STR

% grep STR FILE_NAME > TEMP;more TEMP;rm TEMP
% grep STR FILE_NAME | more	| 管道，把一个命令的输出当做文件交给另一个命令处理
```
```
% find DIR_NAME CONDITION [HANDLE]	默认的handle是print
% find . -name FILE_NAME	-name 表示按名查找，在当前目录（包括子目录）下的FILE_NAME（包括影藏文件）
% find / -name "f*"		使用通配符的时候要加引号，单双都可以（-name 后只能有一个参数！让find去处理通配符，而不是shell）
% find . -name "*.cpp" -exec cp {} DEST_PATH \;		找到文件后复制到DEST_PATH
% find . -name "*.cpp" -exec rm {} ";"			\; 和 ";" 表示-exec 选项的结束。用它本身的含义，而非连接多个命令
```
```
% mkfifo p.pipe		创建管道文件
```
```
% ln /bin/bash <LINK_NAME>	建立硬链接，硬链接要求在同一个物理设备上（无效的跨设备连接）
% ln -s /bin/ls <LINK_NAME>	建立软链接
```
- window下的软链接（快捷方式）保存了很多垃圾数据，linux下的怎非常干净！"Microsoft Windows"这个字符串就占用了好几十兆
- 删除一个硬连接，只是把该硬链接指向的文件的硬链接数-1，只有当硬链接数为0时才真正删除文件。
- 目录不能建硬链接，目录的硬链接数是根据每个目录下的点（.）和点点（..）以及其本身来计算的。
- 硬链接相互平等。软链接不是。
- - -
```
% cp <FILE_NAME> <DEST_FILE_NAME>（路径）	复制文件
% cp -i <FILE_NAME> <DEST_DIR_NAME>		-i有提示（覆盖）(interactive)
% cp -r -i <DIR_NAME_1> [DIR_NAME_2 [...]] <DEST_DIR_NAME>	-r复制目录，当且仅当目录下有文件时才会提示覆盖
	复制一个文件也可改名
```
```
% mv <NAME_1> [NAME_2 [...]] <DEST_DIR_NAME>	移动（多个）文件
% mv <NAME_1> <DEST_DIR_NAME>/<NAME_2>	移动文件并改名，只有移动一个文件的时候才能改名
% mv <NAME_1> <NAME_2>	改名
```
```
% rm -rf 文件名		r:递归删除
% rmdir		删除空目录
% rm -r
	删除用户主目录就无法登录，提示没有这个目录
```
```
% du	显示当前目录（包括子目录）的磁盘使用情况 disk use
% du -k	以K为单位
% du -h	以合适的单位
% du -s	只显示总和

% df	显示磁盘的空闲空间 disk free
% df -h
```
## 四、	通配符
- '*' 匹配任意多个字符（0～n）
	- e.g.
  ```
	% rm <DIR>/*		删除DIR目录下的所有文件或目录
	% ls -l DIR/f*2		列出f打头2结尾的文件或目录
	% rm a*b*c
  ```
	- 通配符由shell解释：	% echo a*b*c
- '?' 匹配任意的一个字符
  ```
	% ls f?
  ```
- []匹配指定范围内的一个字符
	- [b-d]|[1-3]|[135ajm]|[2-46]匹配 b到d|1到3|135ajm|2到4和6 中的任意一个字符（包含边界）
	- 有些系统倒序也匹配 e.g. [4-1]	
  ```
	% mv f[135ajm] DIR_NAME		移动多个文件
  ```

## 五、	网络相关
```
% ping IP	检测和某个主机是否联通，按下Ctrl+c显示统计信息
% ifconfig -a	显示所有网卡ip
```
```
% telnet 127.0.0.1	连接远程服务器（无盘工作站）
% logout	退出，或按ctrl+d
```
```
% ftp IP
	ftp> help
	ftp> help <CMD>
	ftp> cd		进入服务器上的某个目录
	ftp> lcd	进入本地目录
	ftp> ls 或dir
	ftp> !ls	!表示执行本地命令
	ftp> mkdir DIR_NAME
	ftp> put FILE_NAME	上传
	ftp> get FILE_NAME	下载
	ftp> mput FILE_NAME_1 FILE_NAME_2 ...	上传多个文件
	ftp> mget *	下载多个（所有）
	ftp> prompt	打开/关闭交互
	ftp> asc	传输模式变为字符模式（ASCII），跨系统传输需要转变！
	ftp> bin	传输模式变为二进制模式
	ftp> bye	退出
```
```
% write		聊天
% mesg n	不想聊天
```
```
% setup		设置（防火墙et al.）
```
## 六、	Linux环境变量
```
% env	显示环境变量
```
-	$+NAME 表示这个环境变量的值
  -	$PWD
  -	$HOME
  -	$SHELL
  -	$USER
  -	$CC	c编译器
  -	$LANG
```
% PS1='`pwd`$ '	设置命令提示符，为当前完整工作路径
% PS1="[\u@\h \W]\$ "	\u：用户名；\h：主机名；\W：当前目录；\$：$，注意，有空格，加引号
```
- $PATH 为shell指定可执行文件的查找目录
```
% PATH=$PATH:.	包含当前文件路径

% bak=$PATH	备份
% PATH=$PATH:...
% PATH=$bak
```
```
% which cal	查看哪个cal命令被执行
% whereis cal	查看哪里有cal（全部），包括帮助文档的路径
```
```
% alias c=clear	把clear命令取一个别名
% alias pl='ps -l'	有空格，加单引号或双引号
```
## 七、	初始化文件：
- 系统级：/etc
- 用户级：/~
- 不同的shell是不同的，可以用man SHELL_NAME 查看。
	- bash ：
  ```
	% vi ~/.bashrc（shell脚本）
	原则：只加不减
	末尾添加
	PATH=$PATH:. 
	alias pl='ps -l'
	echo "Welcome Back!"
  ```
然后重启终端或 % source ~/.bashrc
