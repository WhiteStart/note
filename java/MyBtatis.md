# MyBatis的优缺点

## 缺点:

- SQL编写工作量较大，对开发人员编写语句有一定要求
- 数据库移植性差，不能随意更换数据库

## 优点:

- SQL写在XML文件里，与程序代码解耦，便于管理
- XML标签支持动态编写SQL
- 消除了JDBC大量冗余代码
- 很好的与数据库兼容(使用JDBC链接数据库)
- 提供了很多第三方插件(分页、逆向工程)



# 什么是半自动化

开发人员需要手动编写SQL，但MyBatis提供了一些工具来简化这一过程



# #{}和${}的区别

- `#{}`是预编译处理，`${}`是字符串替换
- 处理#{}时，会将sql中的#{}替换为?，调用PreparedStatement的set方法来赋值
- 处理`${}`时，就是把`${}`替换成变量的值
- 使用#{}可以防止sql注入，提高系统安全性