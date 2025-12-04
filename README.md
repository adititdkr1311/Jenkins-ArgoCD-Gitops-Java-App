Gitops using argoCD and CICD 

-- K8s cluster setup  
		 - Create K8s cluster 
		 - Create new namespace for argoCD tool 
		 
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
