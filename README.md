# Kubernetes Setup

1. sudo nano /boot/cmdline.txt: Add this text at the end of the line, but don't create any new lines:
	cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
2. sudo nano /etc/apt/sources.list.d/kubernetes.list : add line
	deb http://apt.kubernetes.io/ kubernetes-xenial main
3. curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
4. sudo apt-get update
5. Disable SWAP: 
	sudo dphys-swapfile swapoff
	sudo dphys-swapfile uninstall
	sudo update-rc.d dphys-swapfile remove
	sudo apt purge dphys-swapfile
	sudo free -m
6. sudo apt-get install -qy kubeadm
7. sudo sysctl net.bridge.bridge-nf-call-iptables=1 

	Master Node Setup

	1. sudo kubeadm init --apiserver-advertise-address=<IPAddress>
	2. kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
	3. mkdir -p $HOME/.kube 
	4. sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
	5. sudo chown $(id -u):$(id -g) $HOME/.kube/config
	
	To Join nodes
	
	sudo kubeadm join --token <token> <IPAddress>:6443 --discovery-token-ca-cert-hash <Hash>
	
	Help: Find join command
	
	openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //' 

	DashBoard Setup

	1. sudo apt-get insatll git
	2. git clone https://github.com/skumarvlab/kubernetes
	3. kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
	4. Update IP and apply: kubectl apply -f ./kubernetes/metallb-conf.yaml
    kubectl create -f https://raw.githubusercontent.com/skumarvlab/docker/master/kubernetes-dashboard-arm.yaml
	4. kubectl create serviceaccount dashboard -n default
	5. kubectl create clusterrolebinding dashboard-admin -n default \
  --clusterrole=cluster-admin \
  --serviceaccount=default:dashboard

	Traefik Setup

	1. kubectl apply -f https://raw.githubusercontent.com/skumarvlab/docker/master/alpine/traefik-k8s.yaml
	2. kubectl label node pi01 nginx-controller=traefik
	3. kubectl apply -f https://raw.githubusercontent.com/skumarvlab/docker/master/kubernetes-dashboard-arm-ingress.yaml
