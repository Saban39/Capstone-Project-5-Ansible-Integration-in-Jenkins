aws configure set region eu-west-1
aws eks --region eu-west-1 update-kubeconfig --name my-eks-cluster
kubectl get nodes
aws eks --region eu-west-1 update-kubeconfig --name my-eks-cluster
kubectl get nodes
kubectl apply -f nginx-deployment.yaml
kubectl get service nginx-service
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)        AGE
nginx-service   LoadBalancer   10.0.1.120      aabbccddeeff.elb.eu-west-1.amazonaws.com   80:30498/TCP   2m
kubectl delete -f nginx-deployment.yaml
terraform destroy


terraform init
terraform apply

