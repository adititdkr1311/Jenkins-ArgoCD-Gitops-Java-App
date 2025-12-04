Gitops using argoCD and CICD 

-- Deploy JAVA application to K8s using gitops 
   
   CI = githubActions 
   CD = argoCD 

-- Flow
   source code -> change -> commit -> push -> Trigger github action ->
      checkout + build code + create new image + push new image to dockerhub + update new image name in deployment yaml 
       -> Trigger argoCD -> Deploy new image to K8s cluster 

-- github setup
     > As we have to push the image to dockerhub, we will create secrets in the repository to save dockerhub username and dockerhub password
     > For this go to Settings tab of repository > go to secrets and variables > click on to Actions > Secrets Tab > Repository secrets > click on new repository secret
	 > give name as DOCKERHUB_USERNAME ==> Secret as > your dockerhub username > Click on Add secret
	 > Again  click on new repository secret > give name as DOCKERHUB_TOKEN > Secret as > your dockerhub password> click on add secret
	 > Give the secret names in UPPERCASE, as we use the same secret name in the actions YAML file.
	
-- K8s cluster setup  
		 - install kubernetes on the Ec2 instance spinned using terraform
		 - Steps
		    # sudo su -
			# apt-get update && apt-get install -y curl apt-transport-https
            # sudo mkdir -p /etc/apt/keyrings
            # curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
            # echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
            # sudo apt-get update
            # sudo apt-get install -y containerd.io docker-ce
            # curl -s https://packages.cloud.google.com/apt/doc/yum-key.gpg | apt-key add -
            # echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >/etc/apt/sources.list.d/kubernetes.list
            # apt-get update
            # apt -y install kubeadm kubectl kubelet

           > Execute below set of commands to Initialize Kubernetes Cluster:
               sed -i 's|disabled_plugin|#disabled_plugin|g' /etc/containerd/config.toml 
               service containerd restart
               kubeadm init --ignore-preflight-errors=all
			   
		  > Once Kubernetes Cluster is initialized you can proceed with below commands:
				mkdir -p $HOME/.kube
				sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
				sudo chown $(id -u):$(id -g) $HOME/.kube/config
				kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
				kubectl taint nodes --all node-role.kubernetes.io/control-plane-
				kubectl get nodes	   

		 > We also need to install Helm utility using below set of commands:
				curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get-helm-3 > get_helm.sh
				chmod 700 ./get_helm.sh

         > Now we have to enable root password and root user authentication to proceed with SSH connectivity from GitHub actions pipeline. 
				sed -i 's|PasswordAuthentication no|PasswordAuthentication yes|g' /etc/ssh/sshd_config
				sed -i 's|#PermitRootLogin prohibit-password|PermitRootLogin yes|g' /etc/ssh/sshd_config
				service sshd restart
				passwd root
				ssh root@localhost "date"		(Enter Root user password configured by you in previous command)

          
		 
-- argoCD setup  
		 - apply argocd yaml    --- all the argocd objects will be installed
         - access argocd tool using service. Hence change service type from clusterIP to load balancer 
         - access argoCD using https://<External-IP>
		          username = admin 
			        password = Its a secret to retrive 

-- integrate argoCD with github 
      - Go to ArgoCD settings -> repositories -> Connect repo 
        - create new project 
            - source : path where the templates are 
            - destination : which namespace to spin the K8s objects 		
