Linux文档知识
------
<font face="黑体"  color=blue size=4>**写在前面：**</font> 
Linux文档指的是文件和目录，针对每一个文档，Linux系统定义了三种身份，每种身份又对应着三种权限。

>身份|权限
>:-:|:-:
>owner    group    others|readable    writable    executable
>u, g, o 代表三种身份，a 表示全部|r, w, x 表示三种权限


>文件|目录
>-|-
>针对的是该文件的内容|针对的是该目录下的文件对象
>readable 可读取该文件的实际内容|readable 具有读取目录结构清单的权限，即可以通过ls命令，查询该目录清单。
>writable 可以编辑、新增或者是修改该文件的内容|writable 具有变动该目录结构清单的权限，即可以创建、迁移、删除、更名该目录下的文件。
>executable 有可以被系统执行的权限|executable 具备进入该目录的权限，即可以通过cd命令，转到工作目录。
>备注：具有w权限不可以删除文件，删除文件是目录权限控制的范围！！！记住文件权限针对是文件内容。|备注：从上面可以得出，开放目录给任何人浏览时，至少需要赋予r或x权限。读取目录文件内容，至少需要目录权限x和文件权限r。

<font face="黑体" color=blue size=4>**文档属性：**</font>

>1. d表示目录
>2. -表示文件
>3. l表示连接文件
>4. b表示可随机存取的设备，如U盘等
>5. c表示一次性读取设备，如鼠标、键盘等

<font face="黑体"  color=blue size=4>**Linux的用户和群组：**</font>
在linux中的每个用户必须属于一个组，不能独立于组外。在linux中每个文档有所有者、所在组、其它组的概念。

Linux安全模型|用户概述|用户群组概述
-|:--|-
多任务，多用户的操作系统||
- 使用user和group控制使用者对文件的存储权限。|每个用户（user）都有一个唯一的UserID---UID|每个user都属于一个group，具有唯一的标识符gid
- 用户使用账户和口令登录linux|user的信息存储在/etc/passwd中|group信息存储于/etc/group中
- 每个文件都有owner（创建者），owner属于某个group| 每个usr都有一个home目录|系统会为每一个user关联一个和user同名的group：每个user至少存在于和自己同名的group中（新加user时系统自动创建一个同名的group）；user可以加入其他的group，可以同时属于多个group
- 每个程序都有owner和group|user未经授权将禁止读写执行其他user的文件|同一个group中的成员可以共享其他成员的文件；group不可随便修改，会导致对应关系混乱

<font face="黑体"  color=red size=4>**变更身份：**</font>
<font face="宋体"  color=blue size=3>1. 变更拥有者（owner）</font>

- 位置：`etc/passwd`， 必须是该位置下已存在的帐号。也就是在/etc/passwd中有记录的拥有者才能改变。
- 语法：
```linux
chown [-R] [帐号名称] [文件或目录]
chown [-R] [帐号名称]:[群组名称] [文件或目录]
-R 递归变更，即连同次目录下的所有文件(夹)都要变更。
通过chown改变文件的拥有者和群组。普通用户不能将自己的文件改变成其他的拥有者。其操作权限一般为管理员。
```
- 用法：
```linux
chown lxj test	变更文档test拥有者为lxj
chown lxj:root test	变更文档test群组为root，拥有者为lxj
chown root.users test 变更文档test拥有者为root，群组为users
chown .root test 单独变更文档test群组为root
```

<font face="宋体"  color=blue size=3>2. 变更群组（group）</font>
- 位置：`etc/group`，从这里可以查看到所有群组
- 语法：
```linux
chgrp [-options] [群组名] [文档路径]
关于options，可以通过man chgrp、info chgrp、chgrp --help等命令查询详细用法。
```
- 用法：
```linux
chgrp -R users test 改变test文件夹及其所有子文件(夹)的群组为users。
```
<font face="黑体"  color=red size=4>**变更权限：**</font>
<font face="宋体"  color=blue size=3>1. 符号法</font>
- 分别使用u，g，o来代表三种身份，a表示全部身份；分别使用r、w、x表示三种权限；分别使用+、-、=表示操作行为
- 语法：
```linux
chmod | u g o a | +（加入） -（除去） =（设置） | r w x | 文档路径
```
- 设置权限（=）：
```linux
chmod u=rwx,g=rwx,o=rwx test
或
chmod ugo=rwx test
或
chmod a=rwx test
设置目录test的权限为任何人都可读、写、执行。
```
- 去除权限（-）
```linux
chmod u-x,g-x,o-x test
或
chmod ugo-x test
或
chmod a-x test
去除目录test执行权限
注意：执行权限(x)，对目录而已就是其他用户能否cd test成为工作目录。
```
- 添加权限（+）
```linux
chmod u+x,g+x,o+x test
或
chmod ugo+x test
或
chmod a+x test
添加目录test执行权限
备注：很熟悉吧，如果我们编写完一个shell文件test.sh后，通过chmod a+x test.sh就添加了文件执行权限。
```
<font face="宋体"  color=blue size=3>2. 数字法</font>
顾名思义，就是使用数字来代表权限，r,w,x分别为4,2,1。三种权限累加就可以得出一种身份的权限。

```linux
chmod 777 test
设置目录test的权限为任何人都可读、写、执行。4+2+1~4+2+1~4+2+1
```
```linux
chmod 666 test
设置目录test的权限为任何人都可读、写。4+2~4+2~4+2
```
```linux
chmod 755 test
赋予一个shell文件test.sh可执行权限，拥有者可读、写、执行，群组账号和其他人可读、执行。4+2+1~4+1~4+1
```

<font face="黑体"  color=red size=4>**小结：**</font>
Linux的每个文档可以分别针对三种身份赋予`rwx`权限

- `chown`变更文件拥有者
- `chgrp`命令变更文件群组
- `chmod`命令变更文件权限。