
## 1、 Rancher搭建

### 1.centos搭建docker环境

1. 参考docker安装部分,阿里云的镜像安装

   ```bash
   # step 1: 安装必要的一些系统工具
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   
   # Step 2: 添加软件源信息
   sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   
   # Step 3: 更新并安装 Docker-CE
   sudo yum makecache fast 
   sudo yum -y install docker-ce
   
   # Step 4: 开启Docker服务【要启动起来】
   sudo service docker start
   ```

### 2.docker-machine

1. 官方地址，最新版本的命令

   ```
   https://github.com/docker/machine/releases/
   ```

2. 执行命令，安装docker-machine 

   > 【本步骤可以不用】

   ```bash
   curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && 
   chmod +x /tmp/docker-machine && 
   sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
   ```

3. 查看是否创建成功

   ```bash
   # 查看版本
   docker-machine version
   docker-machine version 0.16.2, build bd45ab13
   ```

4. 内存查看和释放

   ```bash
   free -m
   
   echo 1 > /proc/sys/vm/drop_caches
   echo 2 > /proc/sys/vm/drop_caches
   echo 3 > /proc/sys/vm/drop_caches
   free -m
   ```
### 3.docker-compose

1. 安装
   ```bash
   curl -L https://github.com/docker/compose/releases/download/1.25.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
   
   chmod +x /usr/local/bin/docker-compose
   ```
   
2. 卸载
   
   ```  bash
   sudo rm /usr/local/bin/docker-compose
   ```
   
3. 查看版本
   ```bash
   docker-compose --version
   docker-compose version 1.24.1, build 4667896b
   ```


### 4.启动容器

1. 执行和启动，主master 节点

   ```bash
   docker run -d --restart=unless-stopped \
   >   -p 80:80 -p 443:443 \
   >   -v /host/certs:/container/certs \
   >   -e SSL_CERT_DIR="/container/certs" \
   >   rancher/rancher:latest
   ```
   
2. 主节点注册进来

   ```bash
   #主节点注册进来
   
   sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.8 --server https://192.168.1.207 --token r84gdk8jq7dx56xgrfndbblvpvc69z5b4zcmbht498cp5ttr9llwqv --ca-checksum 6b86d664120d749a8c9ee53cbfb98f966a53d3b4f5ef0ffee7c02016f13cba6c --node-name wehealth-k8s --internal-address 192.168.1.207 --etcd --controlplane --worker
   ```

3. 其他的节点，需要安装docker环境

   > 具体参考2.3 环境准备

4. work节点配置，node1,node2,node3

   > 【注意事项】

   - 此注册的id ，需要根据主节点的UI界面，配置生产出来，并不是通用的

   - 节点名称不能相同，否则重名也会由电商的问题

   - 注册的主机ip，需要配置对，开始就这个配置不对导致了问题

   ```bash
   # node1
   sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.8 --server https://192.168.1.207 --token r84gdk8jq7dx56xgrfndbblvpvc69z5b4zcmbht498cp5ttr9llwqv --ca-checksum 6b86d664120d749a8c9ee53cbfb98f966a53d3b4f5ef0ffee7c02016f13cba6c --node-name work-node1 --internal-address 192.168.1.216 --etcd --controlplane --worker
   
   # node2
   sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.8 --server https://192.168.1.207 --token r84gdk8jq7dx56xgrfndbblvpvc69z5b4zcmbht498cp5ttr9llwqv --ca-checksum 6b86d664120d749a8c9ee53cbfb98f966a53d3b4f5ef0ffee7c02016f13cba6c --node-name work-node2 --internal-address 192.168.1.217 --etcd --controlplane --worker
   
   # node3
   sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.8 --server https://192.168.1.207 --token r84gdk8jq7dx56xgrfndbblvpvc69z5b4zcmbht498cp5ttr9llwqv --ca-checksum 6b86d664120d749a8c9ee53cbfb98f966a53d3b4f5ef0ffee7c02016f13cba6c --node-name work-node3 --internal-address 192.168.1.218 --etcd --controlplane --worker
   ```

### 4.开始使用

1. 本地访问

   ```bash
   192.168.1.207
   ```

2. 安装一下kubectl

   ```bash
   # 先看版本
   $ kubectl version 
   
   # 配置源，aliyun 的
   cat <<EOF > /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   EOF
   
   # 安装
   yum install -y kubectl
   
   # 检查版本kubectl version
   ```

3. 网络插件

   ```
   默认自带组件选项，Flannel
   ```
   
4. 使用过程并不理想，总是出各种问题


## 2、 Docker for Mac/wondows

### 1.安装desktop

1. macOS直接下载安装，见有道云笔记

   ```
   下载镜像iso 文件，安装即可
   文件大概581M
   ```

2. 需注意 2.1.0.1 版本的问题

   > 此版本的 kubernetes 可在设置 国内仓库的情况下，正常启用起来，正常使用；
   >
   > 但是推荐后升级的小版本，反而不能正常，降级回来的；

### 3.dashboard

1. 安装dashboard

   ```bash
   # 删除旧的之前版本，路径下执行的方式
   kubectl delete -f kubernetes-dashboard.yaml
   # 删除之前版本
   kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
   
   docker images
   docker image rm 4640949a39e6
   
   kubectl -n kube-system edit service kubernetes-dashboard
   type: ClusterIP 改成
   type: NodePort
   
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml --force
   ```

2. 运行代理和访问路径

   ```bash
   kubectl proxy
   
   http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
   ```

### 2.token 获取

1. token 获取

   ```bash
   kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token
   
   Name:         namespace-controller-token-fkhhv
   Type:  kubernetes.io/service-account-token
   token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1ma2hodiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6Ijg4Y2VmNGI0LWQ0NTYtMTFlOS1hOTdiLTAyNTAwMDAwMDAwMSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.p6r4Iw0Sk68nP1wRGgn1DeU4BJyV6O0Wb2DYPn7rjVX3Bn15btsyYfJ6o5j8BdoeC-nFXiFYW0jvTj1h76OWRvDWXuTIclypSaf2p2VXjqezop3VOAKoDMSMu6rVym6VhUPQqbL6lguniL2Mh7DfbWTtqPCwjFIuHfxdqvJ43_4lOf2sDUcgyNqwEVCRr2XYx6U8UFwOp8RIhi3z9eIu6_GKCyZRepNet9wdBa1Thun7j8O972zcb16s-6EJMXaduIB1RmF67zjmumGgaK_xj4N4A0NuQ15FsdazS__MWbniGJyifwXptfx3GwjeH1BH4KFWx3k1i43jd7vcPtNxIQ
   ```

2. 配置账号密码的方式

## 2、kubeadm 搭建


### 1、安装要求

1. 安装要求

   - 一台或多台机器，操作系统 CentOS7.x-86_x64
   - 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
   - 集群中所有机器之间网络互通
   - 可以访问外网，需要拉取镜像
   - 禁止swap分区

2. 准备环境

   ```bash
   #关闭防火墙：
   $ systemctl stop firewalld
   $ systemctl disable firewalld
    
   #关闭selinux：
   $ sed -i 's/enforcing/disabled/' /etc/selinux/config 
   $ setenforce 0
    
   #关闭swap：
   $ swapoff -a  $ 临时
   $ vim /etc/fstab  $ 永久
    
   #添加主机名与IP对应关系（记得设置主机名）：
   $ cat /etc/hosts
   192.168.1.62 k8s-master
   192.168.1.64 k8s-node1
   192.168.1.66 k8s-node2
    
   #将桥接的IPv4流量传递到iptables的链：
   $ cat > /etc/sysctl.d/k8s.conf << EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   $ sysctl --system
   ```

### 2、环境配置Ubuntu安装Docker

1. 14.04 16.04 (使用apt-get进行安装)

   ```bash
   # step 1: 安装必要的一些系统工具
   sudo apt-get update
   sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
   # step 2: 安装GPG证书
   curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
   # Step 3: 写入软件源信息
   sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
   # Step 4: 更新并安装 Docker-CE
   sudo apt-get -y update
   sudo apt-get -y install docker-ce
   
   注意：其他注意事项在下面的注释中
   # 安装指定版本的Docker-CE:
   # Step 1: 查找Docker-CE的版本:
   # apt-cache madison docker-ce
   #   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
   #   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
   # Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
   # sudo apt-get -y install docker-ce=[VERSION]
   
   # 通过经典网络、VPC网络内网安装时，用以下命令替换Step 2、Step 3中的命令
   # 经典网络：
   # curl -fsSL http://mirrors.aliyuncs.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
   # sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyuncs.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
   # VPC网络：
   # curl -fsSL http://mirrors.cloud.aliyuncs.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
   # sudo add-apt-repository "deb [arch=amd64] http://mirrors.cloud.aliyuncs.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
   ```


### 3、环境配置Centos安装Docker

1. Centos 7 (使用yum 安装)

   ```bash
   # step 1: 安装必要的一些系统工具
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   # Step 2: 添加软件源信息
   sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   # Step 3: 更新并安装 Docker-CE
   sudo yum makecache fast
   sudo yum -y install docker-ce
   # Step 4: 开启Docker服务
   sudo service docker start
   
   注意：其他注意事项在下面的注释中
   # 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
   # vim /etc/yum.repos.d/docker-ce.repo
   #   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
   #
   # 安装指定版本的Docker-CE:
   # Step 1: 查找Docker-CE的版本:
   # yum list docker-ce.x86_64 --showduplicates | sort -r
   #   Loading mirror speeds from cached hostfile
   #   Loaded plugins: branch, fastestmirror, langpacks
   #   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
   #   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
   #   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
   #   Available Packages
   # Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
   # sudo yum -y install docker-ce-[VERSION]
   # 注意：在某些版本之后，docker-ce安装出现了其他依赖包，如果安装失败的话请关注错误信息。例如 docker-ce 17.03 之后，需要先安装 docker-ce-selinux。
   # yum list docker-ce-selinux- --showduplicates | sort -r
   # sudo yum -y install docker-ce-selinux-[VERSION]
   
   # 通过经典网络、VPC网络内网安装时，用以下命令替换Step 2中的命令
   # 经典网络：
   # sudo yum-config-manager --add-repo http://mirrors.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
   # VPC网络：
   # sudo yum-config-manager --add-repo http://mirrors.could.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
   ```


### 4、启动和验证

1. 启动

   ```bash
   systemctl enable docker && systemctl start docker
   ```

2. 版本

   ```bash
   docker version
   
   # 出现
   Client: Docker Engine - Community
    Version:           19.03.1
    API version:       1.40
    Go version:        go1.12.5
    Git commit:        74b1e89e8a
    Built:             Thu Jul 25 21:21:35 2019
    OS/Arch:           linux/amd64
    Experimental:      false
   
   Server: Docker Engine - Community
    Engine:
     Version:          19.03.1
     API version:      1.40 (minimum version 1.12)
     Go version:       go1.12.5
     Git commit:       74b1e89e8a
     Built:            Thu Jul 25 21:20:09 2019
     OS/Arch:          linux/amd64
     Experimental:     false
    containerd:
     Version:          1.2.6
     GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
    runc:
     Version:          1.0.0-rc8
     GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
    docker-init:
     Version:          0.18.0
     GitCommit:        fec3683
   ```

### 5、准备安装源

1. 镜像源的设置

   ```bash
   # 设置阿里云的源
   cat <<EOF > /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
   enabled=1
   gpgcheck=0
   repo_gpgcheck=0
   gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
           http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   EOF
   ```

2. 如果不能翻墙，方式【一】

   ```bash
   获取当前版本kubeadm 启动需要的镜像
   kubeadm config images list
   
   k8s.gcr.io/kube-apiserver:v1.15.3
   k8s.gcr.io/kube-controller-manager:v1.15.3
   k8s.gcr.io/kube-scheduler:v1.15.3
   k8s.gcr.io/kube-proxy:v1.15.3
   k8s.gcr.io/pause:3.1
   k8s.gcr.io/etcd:3.3.10
   k8s.gcr.io/coredns:1.3.1
   ```

3. 拉取阿里云杭州的镜像到本地

   ```bash
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.3 && \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.3 && \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.3 && \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.3 && \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 && \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10 && \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1 && \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 \
   ```

4. 使用命令放入本地镜像

   ```bash
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.3 k8s.gcr.io/kube-apiserver:v1.15.3 && \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.3 k8s.gcr.io/kube-controller-manager:v1.15.3 && \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.3 k8s.gcr.io/kube-scheduler:v1.15.3 && \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.3 k8s.gcr.io/kube-proxy:v1.15.3 && \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1  k8s.gcr.io/pause:3.1 && \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10 && \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1 && \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1 k8s.gcr.io/pause-amd64:3.1 \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.0 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1 \
   ```

5. 如果不能翻墙，方式【二】，可设置参数

   ```bash
   sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.15.3 --pod-network-cidr=192.168.1.0/16
   ```

   - 有网络说需要修改几处，验证发现不修改反而是正确的

   ```bash
   # 参看这个地址的配置
   # 具体本地文件的地址，ubuntu 在 这个目录下
   cd /etc/systemd/system/kubelet.service.d
   # 网络地址如下
   https://github.com/kubernetes/kubernetes/blob/master/build/rpms/10-kubeadm.conf
   ```
   
   - 不修改，改了又改回来了。

   ```bash
   # Note: This dropin only works with kubeadm and kubelet v1.11+
   [Service]
   Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
   Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
   # This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
   EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
   # This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
   # the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
   EnvironmentFile=-/etc/sysconfig/kubelet
   ExecStart=
   ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
   ```
   
6. 如果修改此文件，需要重新载入

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```
   
   


### 6、kubeadm、kubelet、kubectl

1. **安装kubeadm，kubelet，kubectl**

   ```bash
   apt-get update && apt-get install -y apt-transport-https curl
   curl -s http://packages.faasx.com/google/apt/doc/apt-key.gpg | sudo apt-key add -
   
   cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
   deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
   EOF
   
   #更新
   apt-get update
   # 安装工具
   apt-get install -y kubelet kubeadm kubectl
   apt-mark hold kubelet kubeadm kubectl
   
   #启用 
   systemctl enable kubelet
   ```

2. 升级和加载

   ```bash
   #centos
   yum install -y kubelet-1.15.3 kubeadm-1.15.3 kubectl-1.15.3
   systemctl daemon-reload
   systemctl restart kubelet
   
   #启用 
   systemctl enable kubelet
   ```


### 7、kubeadm init 

1. 初始化

   - 新的版本支持，增加了--image-repository ，可以使用镜像仓库

   ```bash
   sudo kubeadm init \
      	--image-repository registry.aliyuncs.com/google_containers \
      	--kubernetes-version v1.15.3 \
      	--pod-network-cidr=10.244.0.0/16 \
      	--service-cidr=10.1.0.0/16 \
      	--apiserver-advertise-address=183.11.235.36
   ```

   - 出现以下，要等1分钟

   ```bash
   Your Kubernetes control-plane has initialized successfully!
      
   To start using your cluster, you need to run the following as a regular user:
      
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
      
   You should now deploy a pod network to the cluster.
   Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
   https://kubernetes.io/docs/concepts/cluster-administration/addons/
      
   Then you can join any number of worker nodes by running the following on each as root:
      
   kubeadm join 183.11.235.36:6443 --token m02ebg.ltpax11hrj4pl3gk \
          --discovery-token-ca-cert-hash sha256:59cdd87319a6c4d6d32849d7da635a0f725c0e5992e8da42b03b4f20cf1724f3
   ```

2. 如果为出现successfully，则需要重新进行问题排查，重新初始化；

3. 按提示配置kubectl工具

   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

4. 验证是否安装成功

   ```bash
   curl https://127.0.0.1:6443 -k
   {
     "kind": "Status",
     "apiVersion": "v1",
     "metadata": {
   
     },
     "status": "Failure",
     "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
     "reason": "Forbidden",
     "details": {
   
     },
     "code": 403
   }
   
   # 查看pode 
   kubectl get pods --all-namespaces
   
   # 查看集群配置
   kubeadm config view
   
   ```

5. 查看版本

   ```bash
   kubectl get nodes -o wide
   NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
   wehealth   Ready    master   24h   v1.15.3   192.168.1.73   <none>        Ubuntu 16.04.5 LTS   4.4.0-116-generic   docker://19.3.1
   ```

6. 重置慎用

   ```bash
   kubeadm reset
   ifconfig cni0 down && ip link delete cni0
   ifconfig flannel.1 down && ip link delete flannel.1
   rm -rf /var/lib/cni/
   ```

### 8、网络插件(CNI)

1. 配置网络

   ```bash
   sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   
   #确保能够访问到quay.io这个registery。
   
   sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
   ```

### 9、添加节点

1. 按上述的命令，将要加入的节点，加入到网络

   ```bash
   kubeadm join 183.11.235.36:6443 --token m02ebg.ltpax11hrj4pl3gk \
       --discovery-token-ca-cert-hash sha256:59cdd87319a6c4d6d32849d7da635a0f725c0e5992e8da42b03b4f20cf1724f3 \
       --ignore-preflight-errors=Swap
   ```

2. 单独查看token和sha

   ```bash
   # 获取token
   kubeadm token list
   
   # 获取sha256
   openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
   ```

3. 可能出现时间不同步的情况，需要时间同步

### 10、检查集群信息

1. 获取集群信息

   ```bash
   kubectl get cs
   
   NAME                 STATUS    MESSAGE             ERROR
   controller-manager   Healthy   ok
   scheduler            Healthy   ok
   etcd-0               Healthy   {"health":"true"}
   ```

2. 获取节点

   ```bash
   #获取类型为Deployment的资源列表
   kubectl get deployments
   
   #获取类型为Pod的资源列表
   kubectl get pods
   
   #获取类型为Node的资源列表
   kubectl get nodes
   
   NAME       STATUS   ROLES    AGE    VERSION
   wehealth   Ready    master   126m   v1.15.3
   
   # 查看命名空间，节点
   kubectl get pods --all-namespaces
   
   #查看名称为nginx-XXXXXX的Pod的信息
   kubectl describe pod nginx-XXXXXX
   
   #查看名称为nginx-pod-XXXXXXX的Pod内的容器打印的日志
   kubectl logs -f nginx-pod-XXXXXXX
   
   ```
   
3. 查看配置

   ```bash
   # 查看集配置
   
   kubectl get nodes -o wide
   NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
   wehealth   Ready    master   18h   v1.15.3   192.168.1.73   <none>        Ubuntu 16.04.5 LTS   4.4.0-116-generic   docker://19.3.1
   
   kubectl version
   
   kubeadm config view
   ```

4. 在pod中的容器环境内支线命令(和docker exec 命令类似)

   ```bash
   kubectl exec -it my-nginx-86459cfc9f-lhm5p -- /bin/bash
   ```

### 11、dashboard

1. 安装镜像和插件，安装最新的 kubernetes-dashboard-amd64:v1.10.1 版本
   1.10.0 和1.10.1 版本不对应，就不起来的；

   ```bash
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-amd64:v1.5.4 
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-grafana-amd64:v5.0.4 
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-influxdb-amd64:v1.5.2
   
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
   ```

2. 安装配置yarm

   ```bash
   # 下载在安装
   wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
   # 安装
   kubectl create -f kubernetes-dashboard.yaml
   
   #========直接安装=========#
   kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
   # 修改最下面 type节点，为了方便
   # 修改配置
   kubectl -n kube-system edit service kubernetes-dashboard
   type: ClusterIP 改成
   type: NodePort
   
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml --force
   ```

3. 查询配置

   ```bash
   # 查看配置
   kubectl -n kube-system get service kubernetes-dashboard
   
   NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
   kubernetes-dashboard   NodePort   10.107.121.155   <none>        443:30240/TCP   8m6s
   
   # 查看起来的服务端口
   kubectl get services kubernetes-dashboard -n kube-system
   NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
   kubernetes-dashboard   NodePort   10.111.99.144   <none>        443:30832/TCP   2m17s
   
   # 查看pod 运行
   kubectl get pods  -n kube-system | grep dashboard
   kubernetes-dashboard-5f7b999d65-2ttsz    0/1     ImagePullBackOff   0          9m46s
   
   ```

4. 配置最新的版本

   ```bash
   wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
   
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
   
   # 删除
   kubectl delete -f kubernetes-dashboard.yaml
   # 安装
   kubectl create -f kubernetes-dashboard.yaml
   
   # 重新应用
   kubectl apply -f kubernetes-dashboard.yaml --force
   ```

5. 三种方式开

   - 根据id 和端口号打开

   - 代理方式

     ```bash
     $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
     
     kubectl proxy
      
     http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.
     ```

   - 使用用户证书文件的方式

6. 生成令牌

   ```bash
   kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token
   
   Name:         namespace-controller-token-tx6cq
   Type:  kubernetes.io/service-account-token
   token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1xNnF3cyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjI4ODdlNWJmLWRlNzItNGE2OS1hYjY3LWFlZGNkZDM2YWFhMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.BSilZPnbOjaBGTqDsqFOzHlSK1e0hgKPDmx3cvPnVxQHM98WjfxqmOPUl0n4EjDwvwrlNz3lx9G_iZFK-dMwGClT19rstINSB6HRTKFJDABgj5X2FymSq0Bvq8pLstWF2QZIezxggDOSY24Jdadruu0TcnEs-ENCdwWl4j2HFZRhqdG8c1yAp_oQ84YKdQK9kNZ74JzENR-Wcwwp5M7COUYbyWVAL4PFqTbZuIzr41TGQhAaFNU2jC_Z_waAhQIko6GJR0DSUaBRCv9ZNkZTi6QNaDZryJ8C6FpOqUCObrQnQdtWUIQoLjhbjGkDVUYq2rKGwJ0HTFrtnB2HK0eXAQ
   ```

7. 打开UI 界面

   ```bash
   https://183.11.235.36:30832/#!/overview?namespace=default
   ```

### 12、kuboard

1. 安装

   ```bash
   kubectl apply -f https://kuboard.cn/install-script/kuboard.yaml
   ```

2. 卸载

   ```bash
   kubectl delete -f https://kuboard.cn/install-script/kuboard.yaml
   ```

3. 获取token

   - kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')

   ```bash
   Name:         kuboard-user-token-srbch
   Namespace:    kube-system
   Labels:       <none>
   Annotations:  kubernetes.io/service-account.name: kuboard-user
                 kubernetes.io/service-account.uid: bd4c8c07-aebc-485e-8b23-6cc6ed7765ea
   
   Type:  kubernetes.io/service-account-token
   
   Data
   ====
   ca.crt:     1025 bytes
   namespace:  11 bytes
   token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJvYXJkLXVzZXItdG9rZW4tc3JiY2giLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoia3Vib2FyZC11c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYmQ0YzhjMDctYWViYy00ODVlLThiMjMtNmNjNmVkNzc2NWVhIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmt1Ym9hcmQtdXNlciJ9.U1UxV4B2Cx314ZzeKvM4DIFP6t_ch__uo8bMdAG4kQ-czfKV6bgCIncgvxqpGptLRp2LuLkhSeK-OtEGRYqmPuhN2wiA0-wnGOvPh_8Tj6U24-6CLo59COuHUdsaQWOCMkCiAJnjgVCAdv_eBrjgzcXXoj3ZxjDpF1hg-Q31mu8YPNU7ISIz9QmERZKd-pNftW9nu_FS8WnuJeoTmleIktfjp5X81du9IdM2H3_tKdx841Di7YwQGPkitRRB1Gksd1LqM4LTCQCrnsTIsLzFR06Oe9zXAj28dre-Snw--IxT8RdUGJs6pSmsYXhIb3nFgPi2PUemfMy4MvP9QL14OQ
   ```

4. 访问

   ```bash
   Kuboard Service 使用了 NodePort 的方式暴露服务，NodePort 为 32567；您可以按如下方式访问 Kuboard。
   
   http://任意一个Worker节点的IP地址:32567/
   ```

5. 验证

   ```bash
   kubectl get pods  -n kube-system | grep kuboard
   
   kuboard-7bb8d57995-jtbdk                0/1     Pending   0          6m20s
   ```


### 13 、监控组件支持

1. 监控组件 heapster-influxdb、heapster 

   ```bash
   拉取镜像，放入本地仓库
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-amd64:v1.5.4 \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-grafana-amd64:v5.0.4 \
   sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-influxdb-amd64:v1.5.2 
   
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-amd64:v1.5.4 k8s.gcr.io/heapster-amd64:v1.5.4 \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-grafana-amd64:v5.0.4 k8s.gcr.io/heapster-grafana-amd64:v5.0.4 \
   sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-influxdb-amd64:v1.5.2 k8s.gcr.io/heapster-influxdb-amd64:v1.5.2
   
   yaml文件下载
   cd /opt/data/doc
   mkdir heapster
   wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/grafana.yaml
   wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml
   wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml
   wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml
   
   kubectl delete -f .
   kubectl create -f .
   ```
   
2. 监控不出现的问题解决

   ```bash
   # 查看日志
   kubectl logs -f pods/heapster-645c796f6f-glhhw -n kube-system
   
   #github的说明
   https://github.com/opsnull/follow-me-install-kubernetes-cluster/issues/296
   
   在 heapster.yaml 文件的参数，修改为下面的，就可以解决了
   - --source=kubernetes:https://kubernetes.default?inClusterConfig=false&useServiceAccount=true&auth=&kubeletPort=10250&kubeletHttps=true&insecure=true
   - --sink=influxdb:http://monitoring-influxdb.kube-system.svc:8086
   ```

3. ingress cpntroller

   ```bash
   # 在master上执行
   kubectl apply -f https://kuboard.cn/install-script/v1.15.3/nginx-ingress.yaml
   ```

4. github支持的组件安装

   > **kubespher**

   ```bash
   # github
   https://github.com/kubesphere/kubesphere
   # 官网
   https://kubesphere.io/zh-CN/
   ```

5. heml 组件支持

   

## 2 、纯手动搭建

### 1、手动搭建配置

