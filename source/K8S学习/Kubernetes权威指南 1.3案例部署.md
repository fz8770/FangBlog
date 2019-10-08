# Kubernetes权威指南 1.3案例部署

## 环境

```
[root@master1 ~]# kubectl get node -o wide
NAME      STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
master1   Ready    master   91m   v1.15.4   192.168.252.79   <none>        CentOS Linux 7 (Core)   4.4.195-1.el7.elrepo.x86_64   docker://19.3.2
node1     Ready    <none>   89m   v1.15.4   192.168.252.88   <none>        CentOS Linux 7 (Core)   4.4.196-1.el7.elrepo.x86_64   docker://19.3.2
node2     Ready    <none>   81m   v1.15.4   192.168.252.89   <none>        CentOS Linux 7 (Core)   4.4.196-1.el7.elrepo.x86_64   docker://19.3.2
```

## 创建mysql 服务


vim mysql-rc.yaml

```shell
apiVersion: v1
# 副本控制器RC
kind: ReplicationController
metadata:
  # rc的名称, 全局唯一
  name: mysql
spec:
  # 期待创建的pod个数
  replicas: 1
  selector:
    # 选择符合拥有此标签的pod
    app: mysql
  # 根据模板定义的信息创建pod  
  template:
    metadata:
      labels:
        # pod 拥有的标签，对应上边 RC 的 selector
        app: mysql
    # 定义 Pod 细则     
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        # 容器应用监听的端口号
        ports:
        - containerPort: 3306
        # 注入到容器中的环境变量
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
```

vim mysql-svc.yaml

```shell

apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
   - port: 3306
  selector:
    app: mysql

```

开始创建服务

```shell
kubectl apply -f myql-rc.yaml 
kubectl apply -f myql-svc.yaml 
```

## 创建 Tomcat 服务

vim myweb-rc.yml

```shell
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 2
  selector:
    app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        image: kubeguide/tomcat-app:v1
        ports:
        - containerPort: 8080
        env:
          - name: MYSQL_SERVICE_HOST
            value: 'mysql'
          - name: MYSQL_SERVICE_PORT
            value: '3306'
```

上面文件中已用了 MYSQL_SERVICE_HOST、MYSQL_SERVICE_PORT 环境变量，mysql正是上文中定义的 MySQL 服务名。

vim myweb-svc.yml

```shell
apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  type: NodePort
  ports:
   - port: 8080
     nodePort: 30001
  selector:
    app: myweb
~                
```

开始创建服务

```shell
kubectl apply -f myweb-rc.yaml 
kubectl apply -f myweb-svc.yaml 
```

## 验证

```shell
[root@master1 ~]# kubectl get pod,svc -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE   READINESS GATES
pod/mysql-mpqwv   1/1     Running   0          19m   172.32.2.5   node2   <none>           <none>
pod/myweb-5fw65   1/1     Running   0          17m   172.32.2.6   node2   <none>           <none>
pod/myweb-jn5xv   1/1     Running   0          17m   172.32.1.6   node1   <none>           <none>

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE    SELECTOR
service/kubernetes   ClusterIP   172.16.0.1       <none>        443/TCP          102m   <none>
service/mysql        ClusterIP   172.21.225.229   <none>        3306/TCP         39m    app=mysql
service/myweb        NodePort    172.25.154.78    <none>        8080:30001/TCP   32m    app=myweb

```

浏览器访问 http://192.168.252.79:30001/demo/ 验证

## 参考文章

https://blog.csdn.net/wo18237095579/article/details/89376877