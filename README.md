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

4、完整*Deployment*

```
apiVersion: v1
kind: Namespace
metadata:
  name: mysql
---

apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pwd
  namespace: mysql
data:
  mysql-root-pwd: MTIzNDU2
  mysql-app1-user-pwd: MTIzNDU2
  mysql-test1-user-pwd: MTIzNDU2

---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
  namespace: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pwd
              key: mysql-root-pwd
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pwd
              key: mysql-app1-user-pwd
        - name: MYSQL_USER
          value: app1
        ports:
        - name: mysql-port
          containerPort: 3306
          command:
            - /bin/sh
            - "-c"
            - MYSQL_PWD="${MYSQL_ROOT_PASSWORD}"
            - mysql -h 127.0.0.1 -u root -e "SELECT 1"
          initialDelaySeconds: 30
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - "-c"
            - MYSQL_PWD="${MYSQL_ROOT_PASSWORD}"
            - mysql -h 127.0.0.1 -u root -e "SELECT 1"
          initialDelaySeconds: 10
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 3
        volumeMounts:
        - mountPath: /etc/mysql/conf.d/
          name: mysql-config
        - mountPath: /var/lib/mysql 
          name: mysql-data
      volumes:
      - name: mysql-config
        hostPath:
          path: /usr/iqeq/mysql/conf/
      - name: mysql-data
        hostPath:
          path: /usr/iqeq/mysql/data
          
---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql
  name: mysql
  namespace: mysql
spec:
  type: NodePort
  ports:
  - name: mysql-port
    port: 3306
    nodePort: 23306
    protocol: TCP
    targetPort: 3306
  selector:
    app: mysql
```

5、部署
```
  kubectl apply -f mysql.yaml
```
