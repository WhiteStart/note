# 文件权限显示

```
-  ---  ---  ---
d	 rwx  rw-  r--

1.代表文件类型
  - 文件
  d 文件夹
  l 表示链接

2.当前用户具有该文件的权限
	r 读(read)
	w 写(write)
	x 执行(excute)
	
3.当前组内其他用户具有该文件的权限
	r
	w
	x

4.其他组的用户具有该文件的权限
	r
	w
	x
```

# chmod命令

| u    | user   | 文件所有者       |
| ---- | ------ | ---------------- |
| g    | group  | 文件所有者所在组 |
| o    | others | 所有其他用户     |
| a    | all    | 所有用户         |

```markdown
# u增加读权限
chmod u + r file.txt

# u和g取消执行权限
chmod ug - x file.txt

# 
chmod 
```

```
chmod 命令也可以使用八进制来指定权限

chmod (user 文件所有者)(group 用户组)(other 其他用户) {file_name}
```

| #    | 权限       | rwx  | 二进制 |
| ---- | ---------- | ---- | ------ |
| 7    | 读+写+执行 | rwx  | 111    |
| ...  |            |      |        |
| 4    | 只读       | r--  | 100    |
| ...  |            |      |        |
| 1    | 只执行     | --x  | 001    |
| 0    | 无         | ---  | 000    |

```
chmod 751 file.txt

文件所有者拥有7，即rwx
组内用户拥有5， 即r-x
其他用户拥有1，即--x
```


