# aws-custom-k8s-installation

--------------------------------------------

AWS K8s custom cluster creation on US EAST 1

--------------------------------------------



#1.  Create a lowest powered instance on AWS.

#2.  Create IAM user 'kops' with group-name 'kops', capture access key and secret.

#3.  Kops user needs the following IAM permissions:

	 - AmazonEC2FullAccess

	 - AmazonRoute53FullAccess

	 - AmazonS3FullAccess

	 - IAMFullAccess

	 - AmazonVPCFullAccess





#4.  On the bootstrap instance, install kops and kubectl visiting the following link:

	 - https://github.com/kubernetes/kops/blob/master/docs/install.md and run the following

	 - aws configure # Use your new access and secret key here

	 - aws iam list-users # confirm a list of users

#5.  Create S3 bucket 'moneyball-k8s-state-storage', block public acess which is default

#6.  export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)

#7.  export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)

#8.  export NAME=fleetman.k8s.local

#9.  export KOPS_STATE_STORE=s3://moneyball-k8s-state-storage #above bucket name

#10. Check availability zones with the following command:

	 - aws ec2 describe-availability-zones --region us-east-1

#11. kops create cluster --zones us-east-1a,us-east-1b,us-east-1c,us-east-1d,us-east-1e,us-east-1f ${NAME}

#12. ssh-keygen -b 2048 -t rsa -f ~/.ssh/id

#13. kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub

#14. Check default cluster configuration

	 - kops edit cluster ${NAME}

#15. Exit vi editor and set something friendly

	 - export EDITOR=nano

#16. Check the configuration of EC2 nodes inside the cluster, and edit the instance groups via following

	 - kops edit ig nodes --name ${NAME}

	 - normally we would put maxSize = 5 and minSize = 3 as a starting point.

#17. Confirm the type of nodes and configuration via the following command

	 - kops get ig --name ${NAME}

#18. If you want to edit any instance group, copy the name master or nodes and hit the following:

	 - kops edit ig master-us-east1a --name ${NAME} #if that is the name

#19. To actually start running the K8s cluster in cloud, hit the command:

	 - kops update cluster ${NAME} --yes

#20. Run 'kops validate cluster' to see if the cluster is live or not, takes a few mins, keep hitting this command.

#21. You should see, 'Your cluster ${NAME} is ready'. This is just the cluster, nothing deployed yet.

#22. 'kubectl get all' should list a default kubernetes service running.

#23. Apply all the YAML files with command 'kubectl apply -f .' or a single file xxx.yaml replacing dot.

#24. To delete cluster run 'kubectl delete cluster --name ${NAME} --yes', do not stop/delete the bootstrap instance until all the resources are deleted.

#25. Restarting cluster

	 - if you are not logged into the bootstrap instance, run the following once again

	 - export NAME=fleetman.k8s.local

	 - export KOPS_STATE_STORE=s3://bucket-name

	 - kops create cluster --zones us-east-1a,us-east-1b,us-east-1c,us-east-1d,us-east-1e,us-east-1f ${NAME}

	 - Edit InstanceGroup via running 'kops edit ig nodes --name ${NAME}', set min/max 3 and 5 in our example.

	 - kops update cluster ${NAME} --yes

	 - Apply the YAML files to create all your pods and services.



#26. To view the logs:

	 - kubectl logs name_of_pod



#27. To go inside the running container:

	 - kubectl exec -it name_of_pod bash # bash can be replaced with sh



#28. To install Kibana log monitoring tool, you have to install ELK stack, by default the following installs the stack in 'kube-system' namespace

	 - kubectl apply -f fluentd-config.yaml # we are using fluentd in our case, you can use logstash as well. This will be installed on all available nodes.

	 - kubectl apply -f elk_stack.yaml

#29. Check the Kibana logging service

	 - kubectl describe svc kibana-logging -n kube-system # note the load balancer created and port, and check against AWS LoadBalancers page.

	 - In browser go to, load balancer url with port 5601 to see Kibana portal



#30. For Grafana to monitor cluster health metrics

	 - Install helm, via https://helm.sh, github latest release for linux.

	 - https://get.helm.sh/helm-v2.14.3-linux-amd64.tar.gz # this link could be different

	 - On bootstrap instance, 'wget https://get.helm.sh/helm-v2.14.3-linux-amd64.tar.gz'

	 - Run 'tar zxvf helm-v2.14.3-linux-amd64.tar.gz'

	 - sudo mv linux-amd64/helm /usr/local/bin/

	 - clean/remove unzip and downloaded file to make it look tidy, coz we don't need these files anymore.

	 - helm version # this will probably hang, gives the client version. Hangs because it is trying to find the server version. It is running to run kubectl command in the background trying to connect running pod in the cluster. Hit 'CTRL + C' to exit and continue the following.

	 - helm init # This runs a tiller pod if you visit 'kubectl get po -n kube-system'

	 - Run again 'helm versioin' should show both client and server versions.

	 - helm repo update # updates all the local indexes connecting to remote repository



 #31. Tiller pod has been installed by default into the kube-system namespace, and this pod does not have the correct access privileges to allow it to START UP NEW pods in the default namespace. To fix this access privilege, run the following before  you do any helm install packages.

 	 - kubectl create serviceaccount --namespace kube-system tiller

	 - kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller



	 - kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 



#32. Check helm installations

	 - helm ls



#33. Delete helm installation

	 - helm delete --purge installation_name



#34. helm stable charts page from github

	 - helm install --name monitoring --namespace monitoring stable/prometheus-operator

	 - kubectl get all -n monitoring # check the grafana service



#35. 

























