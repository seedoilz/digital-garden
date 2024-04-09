---
aliases: 
title: 集群SSH免密配置
date created: 2024-04-09 14:04:00
date modified: 2024-04-09 20:04:46
tags:
  - code/snippet
---
## 脚本编写
### xsync
>循环复制文件到所有节点的相同目录下
```shell
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
	echo Not Enough Arguement!
	exit;
fi
#2. 遍历集群所有机器
for host in hadoop1 hadoop2 hadoop3
do
	echo ==================== $host ====================
	#3. 遍历所有目录，挨个发送
	for file in $@
	do
		#4. 判断文件是否存在
		if [ -e $file ]
			then
				#5. 获取父目录
				pdir=$(cd -P $(dirname $file); pwd)
				#6. 获取当前文件的名称
				fname=$(basename $file)
				ssh $host "mkdir -p $pdir"
				rsync -av $pdir/$fname $host:$pdir
			else
				echo $file does not exists!
		fi
	done
done

# 修改脚本 xsync 具有执行权限
chmod +x xsync
# 将脚本复制到/bin 中，以便全局调用
sudo cp xsync /bin/
```
### jpsall
```shell
#!/bin/bash
for host in hadoop1 hadoop2 hadoop3
	do
	echo =============== $host ===============
	ssh $host jps
done

# 保存后退出，然后赋予脚本执行权限
chmod +x jpsall
# 分发/home/atguigu/bin 目录，保证自定义脚本在三台机器上都可以使用
xsync /home/atguigu/bin/
```
## 免密登陆
```shell
# 在hadoop1上生成公钥和私钥
ssh-keygen -t rsa
# 将公钥拷贝到要免密登录的目标机器上
ssh-copy-id hadoop1
ssh-copy-id hadoop2
ssh-copy-id hadoop3
```
> [!Warning] 注意
> 其他节点也需要相同的方法配置一遍
