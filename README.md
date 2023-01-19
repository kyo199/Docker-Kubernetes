	# 스왑 off,주석(vi /etc/fstab)
		sudo swapoff -a
		vi /etc/fstab -> swap 부분 # 로 주석 ( #/swap.img       none    swap    sw      0       0 )
		
	#설정
		sudo modprobe br_netfilter
		sudo modprobe overlay
		
		
		cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
		overlay
		br_netfilter
		EOF
		
		sudo cat <<EOF | sudo tee -a /etc/sysctl.d/99-kubernetes-cri.conf
		net.bridge.bridge-nf-call-iptables  = 1
		net.ipv4.ip_forward                 = 1
		net.bridge.bridge-nf-call-ip6tables = 1
		EOF
		
		sudo sysctl --system
		
		# cri-o 버전 지정
		OS=xUbuntu_20.04
		VERSION=1.25
		
		# Add Kubic Repo
		echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" | \
		sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
		
		# Import Public Key
		curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | \
		sudo apt-key add -
		
		# Add CRI Repo
		echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" | \
		sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
		
		# Import Public Key
		curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | \
		sudo apt-key add -
		
		# cri-o 설치
		sudo apt update
		sudo apt install cri-o cri-o-runc cri-tools -y
		
		sudo systemctl enable crio.service
		sudo systemctl start crio.service
		sudo crictl info
		
		
		# kuectl kubeadm kubelet 설치전에 특정버전 (cri-o 1.25에 맞추어 설치)
		sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
		echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
		sudo apt-get update
		sudo apt-get install -y kubelet kubeadm kubectl (특정버전 맞추어sudo apt-get install kubelet=1.25.4-00 kubeadm=1.25.4-00 kubectl=1.25.4-00 )
		sudo apt-mark hold kubelet kubeadm kubectl
		
		
		# kube init
		sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --control-plane-endpoint=10.10.211.120:6443 --upload-certs
		Ex) sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=10.10.211.151
	
	
		
		# 실패시 아래명령어
		sudo ufw disable
            sudo kubeadm reset


# kube init 명령어는 master에서만 사용하며, node에서는 kube init 항목 대신 master kube init시 발생된 kubeadmin join 항목을 복사해서 붙여넣기 하면 된다.
ex) kubeadm join 10.10.211.151:6443 --token 38dtau.bwula7qrlth786zk \
        --discovery-token-ca-cert-hash sha256:d9970ca63dfa83b29349b9bc51bc22bf89cf9e09948c6d6a1ae27552eccb1599![image](https://user-images.githubusercontent.com/14196841/213378312-adeff219-5b45-46ab-a267-3641b573ea2f.png)

# 마스터 노드만 생성되었을 시
root@master:~# kubectl get nodes
NAME     STATUS   ROLES           AGE   VERSION
master   Ready    control-plane   47s   v1.25.4

# 워커 노드 생성을 kubeadmin join을 했을 시
root@master:~# kubectl get nodes
NAME     STATUS   ROLES           AGE     VERSION
master   Ready    control-plane   25m     v1.25.4
node1    Ready    <none>          4m31s   v1.25.4
	
root@master:~# kubectl get nodes
NAME     STATUS   ROLES           AGE   VERSION
master   Ready    control-plane   32m   v1.25.4
node1    Ready    <none>          11m   v1.25.4
node2    Ready    <none>          55s   v1.25.4
