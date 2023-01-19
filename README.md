	# 스왑 off,주석
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
