# Qwiklabs Implement DevOps in Google Cloud Challenge Lab

# Task 1: Configure a Jenkins pipeline for continuous deployment to Kubernetes Engine

Cloud Shell                                         
-> gcloud config set compute/zone us-east1-b                                               

-> git clone https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/sample-app                                     

-> gcloud container clusters get-credentials jenkins-cd                                                                          

-> kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)                                                    

-> helm repo add stable https://kubernetes-charts.storage.googleapis.com/                                                                      
-> helm repo update                             
-> helm install cd stable/jenkins                                                       

-> export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")                                 

-> kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &                                             

-> printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo                                                                   

You will get password                               
Go to web preview -> Preview on port 8080                   
 
Username : admin                                          
Password : [copied from previous command]                               

Cloud Shell                                        
-> cd sample-app/                            
-> kubectl create ns production                                
-> kubectl apply -f k8s/production -n production                             
-> kubectl apply -f k8s/canary -n production                         
-> kubectl apply -f k8s/services -n production                         
-> kubectl get svc                                     
-> kubectl get services gceme-frontend -n production                                        

Go to Jekins Dashboard                                      
Left Menu->Manage Jenkins->Manage credentials                       
Click on Jenkins -> Click on Global credentials(unrestricted)                                            
Left Menu->Add credentials                                
Kind : Google Service Account from metadata                                  
Project Name : auto generated                                   
OK                               

Jenkins Home page                  
New Item                           
Name : sample-app                              
Multibranch Pipeline                              
OK                                    

Branch Sources : git                                            
Project Repository :https://source.developers.google.com/p/[PROJECT_ID]/r/sample-app                                
"Note : replace [PROJECT_ID] your lab project ID"                                    
Credentials : Select your project ID                                   

In Scan Multibranch Pipeline Triggers                                             
Check Periodically if not otherwise run                                  
interval : 1 minute                                    

Save                               

git init                                    
git config credential.helper gcloud.sh                                                                        
git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/sample-app                                         
git config --global user.email "<user email>"                                       
git config --global user.name "<user name>"                                 
git add .                                          
git commit -m "initial commit"                   
git push origin master                                       

Wait until full process is complete                                               
To check click on Status on left menu                                      

Check my Progress                                              

# Task 2: Push an update to the application to a development branch
Cloud Shell                            
-> git checkout -b new-feature                                
-> vi html.go                       

->i                             
change both lines that contains the word blue to orange                            
->ESC key                                      
->:wq                      

-> vi main.go                              
-> i                                       
change the version number to "2.0.0"                                   
->ESC key                                  
->:wq                                

-> git add Jenkinsfile html.go main.go                           
-> git commit -m "Version 2.0.0"                              
-> git push origin new-feature                                

Go to Jenkins Dashboard -> Click on sample-app ->Notice new item is being created                                                

-> kubectl proxy &                                                                    

-> Starting to serve on 127.0.0.1:8001 [auto-generated command]                                           

-> curl http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version                                                  

-> kubectl get service gceme-frontend -n production                                                              
Copy External IP and paste in new tab                                     

-> git checkout -b canary                                              
-> git push origin canary                                             
-> export FRONTEND_SERVICE_IP=$(kubectl get -o \                                      
      >jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)                    
-> git checkout master                                                                                       
-> git push origin master                                     

-> export FRONTEND_SERVICE_IP=$(kubectl get -o \                                      
          >jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)                                   
-> while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done                                          


Check my Progress                                                                     

# Task 3:Push a Canary deployment to the production namespace                                                 

Cloud Shell                                                          

-> git checkout master                                                  
-> git merge canary                                            
-> git push origin master                                       

Check my Progress                                                  

# Task 4: Promote the Canary Deployment to production

Cloud Shell                                                       

->export FRONTEND_SERVICE_IP=$(kubectl get -o \                                         

>jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)                                    

->while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done                                 

Check my Progress                                                           

# Congratulations! you have successfully completed this challenge lab
