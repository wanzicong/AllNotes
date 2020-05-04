## 遇到的问题

### 1.1 ：将另外一个厂库的代码推到别的远程仓库

解决办法: 

（1） 建立连接之后（git remote add origin https://github.com/wanzicong/springdatajpa-study.git ）

​        git pull  合并厂库

​		git push -m  origin master

  (2) git push -f origin master 强行将本地厂库替换为远程厂库，取而代之	



