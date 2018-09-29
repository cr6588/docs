kubelet为什么关闭swap

kubernetes的想法是将实例紧密包装到尽可能接近100％。 所有的部署应该与CPU /内存限制固定在一起。 所以如果调度程序发送一个pod到一台机器，它不应该使用交换。 设计者不想交换，因为它会减慢速度。所以关闭swap主要是为了性能考虑。
当然为了一些节省资源的场景，比如运行容器数量较多，可添加kubelet参数 --fail-swap-on=false来解决。

关闭swap
swapoff -a