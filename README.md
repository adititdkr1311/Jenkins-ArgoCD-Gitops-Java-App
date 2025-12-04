Gitops using argoCD and CICD 

-- Deploy JAVA application to K8s using gitops 
   
   CI = githubActions 
   CD = argoCD 

-- Flow
   source code -> change -> commit -> push -> Trigger github action ->
      checkout + build code + create new image + push new image to dockerhub + update new image name in deployment yaml 
       -> Trigger argoCD -> Deploy new image to K8s cluster 

	
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
