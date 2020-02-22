# Kubernetes

# Network Setup:

1. Map all domains that will be used for your applications on your computer “hosts” file to nodes: sudo nano /etc/hosts

192.168.1.221 application.internal.mydomain.com

192.168.1.222 dashboard.internal.mydomain.com
192.168.1.221 dashboard.external.mydomain.com

192.168.1.222 traefik.internal.mydomain.com
192.168.1.221 traefik.external.mydomain.com


# Kubernetes Setup
1. sudo nano /boot/cmdline.txt: Add this text at the end of the line, but don't create any new lines:
	cgroup_enable=cpuset cgroup_memory=1
2. sudo nano /etc/apt/sources.list.d/kubernetes.list : add line
	deb http://apt.kubernetes.io/ kubernetes-xenial main
3. sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
4. sudo apt-get update
5. Disable SWAP: 
	sudo dphys-swapfile swapoff
	sudo dphys-swapfile uninstall
	sudo update-rc.d dphys-swapfile remove

	sudo apt purge dphys-swapfile
	sudo free -m
6. sudo apt-get install -qy kubeadm
7. sudo sysctl net.bridge.bridge-nf-call-iptables=1 
8. sudo apt autoremove

	Master Node Setup

	a. sudo kubeadm init --token-ttl=0 --apiserver-advertise-address=<IPAddress> | sudo kubeadm init --token-ttl=0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<IPAddress>
	b. mkdir -p $HOME/.kube 
	c. sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
	d. sudo chown $(id -u):$(id -g) $HOME/.kube/config
	e. kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')" | kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
	f. kubectl taint nodes --all node-role.kubernetes.io/master-
	g. pscp pi@192.168.1.121:/home/pi/.kube/config C:\Private\skumarvlab\private\Lab\pi21.config
	h. Create Envirnament Variable KUBECONFIG=E:\skumarvlab\lab\pi21.config
	i. https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe

	To Join nodes

	sudo kubeadm join --token <token> <IPAddress>:6443 --discovery-token-ca-cert-hash <Hash>
	
	Help: Find join command
	
	openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //' 


	Get Configs
	a. sudo apt-get install git
	b. git clone https://github.com/skumarvlab/kubernetes


	DashBoard Setup
	a. kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml
	b. kubectl create serviceaccount dashboard -n default
	c. kubectl create -f ./kubernetes/dashboard/service-account.yaml
	d. kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
	e. Access at: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/overview?namespace=_all

	LoadBalancer
	a. kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.1/manifests/metallb.yaml
	b. Update IP to master's IP and apply: kubectl apply -f ./kubernetes/metallb/metallb-conf.yaml



Helpfull Commands:

1. kubectl get nodes
2. kubectl get pods --all-namespaces
3. kubectl get services --all-namespaces
4. kubectl proxy 