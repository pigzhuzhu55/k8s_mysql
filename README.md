# 基于k8s部署的mysql

## Installation 

1、 用户密码等敏感数据以*Secret*加密保存，以下脚本生成密码（如123456）的加密串，用于后面设置mysql数据库密码

```linux
# echo -n 123456 | base64
MTIzNDU2
```

*root*、*app*1、*test*用户，将*3*个用户的密码加密保存到k8s的保密字典里面，如下

```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pwd
  namespace: mysql
data:
  mysql-root-pwd: MTIzNDU2
  mysql-app1-user-pwd: MTIzNDU2
  mysql-test1-user-pwd: MTIzNDU2
```

2、配置文件在conf/my.cnf

3、TODO　暂时没考虑做初始化建库脚本的命令
 
4、部署
```
  kubectl apply -f mysql.yaml
```
