# Kubernetes Setup

1. sudo nano /etc/apt/sources.list.d/kubernetes.list : add line
	deb http://apt.kubernetes.io/ kubernetes-xenial main
2. sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
3. sudo apt-get update
4. Disable SWAP: 
	sudo dphys-swapfile swapoff
	sudo dphys-swapfile uninstall
	sudo update-rc.d dphys-swapfile remove
	sudo apt purge dphys-swapfile
	sudo free -m
5. sudo reboot now
6. sudo apt-get install -qy kubeadm
7. sudo sysctl net.bridge.bridge-nf-call-iptables=1 
8. sudo apt autoremove

	Master Node Setup

	a. sudo kubeadm init --token-ttl=0 --pod-network-cidr=10.244.0.0/16
	b. mkdir -p $HOME/.kube 
	c. sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
	d. sudo chown $(id -u):$(id -g) $HOME/.kube/config
	e. kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.244.0.0/16" 
	f. kubectl taint nodes --all node-role.kubernetes.io/master-
	g. pscp pi@192.168.1.221:/home/pi/.kube/config C:\Private\skumarvlab\private\Lab\pi21.config
	h. Create Envirnament Variable KUBECONFIG=C:\Private\skumarvlab\private\Lab\pi21.config
	i. https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe
	j. Add master node lable: kubectl label nodes pi21 disktype=master

	To Join nodes

	sudo kubeadm join --token <token> <IPAddress>:6443 --discovery-token-ca-cert-hash <Hash>
	
	Help: Find join command
	
	openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //' 


	DashBoard Setup
	a. kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
	b. kubectl create -f https://raw.githubusercontent.com/skumarvlab/kubernetes/master/dashboard/service-account.yaml
	c. kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
	d. Access at: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/overview?namespace=_all

	MetalLb
	a. kubectl apply -f  https://raw.githubusercontent.com/skumarvlab/kubernetes/master/metallb/metallb.yaml
	b. kubectl apply -f https://raw.githubusercontent.com/skumarvlab/kubernetes/master/metallb/metallb-config.yaml

	Helm Setup
	a. wget https://get.helm.sh/helm-v3.2.0-linux-arm.tar.gz
	b. tar xvzf helm-v3.2.0-linux-arm.tar.gz
	c. sudo mv linux-arm/helm /usr/local/bin/helm
	d. rm -rf linux-arm
	e. kubectl -n kube-system create serviceaccount tiller && kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller && helm init --service-account tiller --tiller-image=jessestuart/tiller:v2.9.1 --override spec.selector.matchLabels.'name'='tiller',spec.selector.matchLabels.'app'='helm' --history-max 10 --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' | kubectl apply -f -

	Ingress
	a. kubectl apply -f https://raw.githubusercontent.com/skumarvlab/kubernetes/master/traefik/traefik.yaml

Helpful Commands:

1.  kubectl get nodes -o wide
2.  kubectl get pods -o wide --all-namespaces
3.  kubectl get services -o wide --all-namespaces
4.  kubectl proxy 
5.  sudo rm -rf $HOME/.kube/config && sudo kubeadm reset
6.  sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
7.  sudo ip link del flannel.1
8.  sudo ip link del weave
9.  sudo rm -rf /etc/kubernetes/
10. docker image prune -a