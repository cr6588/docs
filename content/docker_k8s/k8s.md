[安装多版本docker](http://blog.51cto.com/11887934/2050590)

kubelet为什么关闭swap

kubernetes的想法是将实例紧密包装到尽可能接近100％。 所有的部署应该与CPU /内存限制固定在一起。 所以如果调度程序发送一个pod到一台机器，它不应该使用交换。 设计者不想交换，因为它会减慢速度。所以关闭swap主要是为了性能考虑。
当然为了一些节省资源的场景，比如运行容器数量较多，可添加kubelet参数 --fail-swap-on=false来解决。

关闭swap
swapoff -a

https://www.cnblogs.com/crysmile/p/9648406.html
1、关闭selinux、firewalld。
2、开启内核转发。
3、关闭swap交换分区
4、master免密钥登录所有node节点
5、所有节点配置ntp时间同步服务，保证节点时间一致。
6、加载ipvs相关模块

We have a Service called kubernetes that is created by default when minikube starts the cluster. To create a new service and expose it to external traffic we’ll use the expose command with NodePort as parameter (minikube does not support the LoadBalancer option yet).

kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080

kubectl get services

Now we can test that the app is exposed outside of the cluster using curl, the IP of the Node and the externally exposed port:

curl $(minikube ip):$NODE_PORT

kubectl delete service -l run=kubernetes-bootcamp


kubectl get deployments
##伸缩pod数量，默认就是负载均衡的
kubectl scale deployments/kubernetes-bootcamp --replicas=4
kubectl get pods -o wide
#修改镜像，ocatalin/kubernetes-bootcamp:v2新的镜像
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2

##回滚操作
kubectl rollout undo deployments/kubernetes-bootcamp

##Cleaning up
kubectl delete services my-service
kubectl delete deployment hello-world
##Exposing an External IP Address to Access an Application in a Cluster
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
kubectl get services my-service

    [root@cr6588 home]# kubectl get services
    NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    kubernetes            ClusterIP      10.96.0.1       <none>        443/TCP          23h
    kubernetes-bootcamp   NodePort       10.110.244.64   <none>        8080:32248/TCP   17h
    my-service            LoadBalancer   10.100.17.78    <pending>     8079:31381/TCP   18m
EXTERNAL-IP一直是pending猜测都是内网的机器？
