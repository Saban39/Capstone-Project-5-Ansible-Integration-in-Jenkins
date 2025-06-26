#### This project is for the Devops bootcamp exercise for 
#### "Configuration Management with Ansible"1002  rm -rf .git\n
 1003  git init
 1004  git add .\ngit commit -m "Initial commit"\n
 1005  git remote add origin https://github.com/Saban39/ansible_exercises.git\n
 1006  hist
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercise
MY NEXUS
http://85.215.45.206:8081/repository/my_test_repo/



curl -v -u sg:seherim01 --upload-file build/libs/build-tools-exercises-1.0-SNAPSHOT.jar \
  "http://85.215.45.206:8081/repository/my_test_repo/com/example/build-tools-exercises/1.0-SNAPSHOT/build-tools-exercises-1.0-SNAPSHOT.jar"


  18.196.13.159 

  ssh ubuntu@18.196.13.159 -i ansible-ssh-key.pem 



  ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem aws_region=eu-central-1 key_name=ansible-ssh-key subnet_id=subnet-0e9565afb5dd37baf ami_id=ami-02003f9f0fde924ea ssh_user=ubuntu"


  ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=ubuntu aws_region=eu-central-1"


  ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem aws_region=eu-central-1 key_name=ansible-ssh-key subnet_id=subnet-0e9565afb5dd37baf ami_id=ami-092ff8e60e2d51e19 ssh_user=ec2-user"


  ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=amazon-linux aws_region=eu-central-1"


  command for 5:
  ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem aws_region=eu-central-1 key_name=ansible-ssh-key subnet_id=subnet-0e9565afb5dd37baf ami_id=ami-02003f9f0fde924ea ssh_user=ubuntu"



1241  ansible-playbook 7-deploy-on-k8s.yaml
 1242  ansible-playbook 7-deploy-on-k8s.yaml
 1243  kubectl get svc
 1244  kubectl get loadbalancer
 1245  kubectl get svc -n ingress
 1246  kubectl get svc -n ingress
 1247  kubectl get pods
 1248  kubectl logs java-app-deployment-66c7f44df4-vb5sj\n
 1249  history | grep docker 
 1250  docker build tag java-mysql-app .
 1251  docker build -t java-mysql-app .
 1252  docker tag java-mysql-app sg1905/demo-app/java-mysql-app
 1253  docker push sg1905/demo-app/java-mysql-app
 1254  docker login 
 1255  docker push sg1905/demo-app/java-mysql-app
 1256  docker push sg1905/demo-app:java-mysql-app

  ansible-playbook -i hosts-jenkins-server 5-install-jenkins-docker.yaml --extra-vars "aws_region=eu-central-1"



  1267  docker build -t java-mysql-app .
 1268  docker images
 1269  docker build -t java-mysql-app .
 1270  docker images
 1271  docker build -t java-mysql-app-2 .
 1272  docekr images
 1273  docker images
 1274  docker tag java-mysql-app-2 sg1905/demo-app/java-mysql-app
 1275  docker push sg1905/demo-app/java-mysql-app
 1276  docker push sg1905/demo-app:java-mysql-app
 1277  docker images
 1278  docker push sg1905/demo-app/java-mysql-app-2
 1279  docker push java-mysql-app-2
 1280  docker push sg1905/demo-app:java-mysql-app-2
 1281  docker tag java-mysql-app-2 sg1905/demo-app:java-mysql-app
 1282  docker push sg1905/demo-app:java-mysql-app



 kubectl delete all --all -n ingress-nginx


View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/4hx775fyt6dei6i1gir499dtk
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% docker tag  java-mysql-app java-mysql-app  
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% docker tag  java-mysql-app java-mysql-app  
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% docker push  sg1905/demo-app:java-mysql-app
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% docker tag  java-mysql-app sg1905/demo-app:java-mysql-app
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% docker push  sg1905/demo-app:java-mysql-app   
The push refers to repository [docker.io/sg1905/demo-app]
5f70bf18a086: Layer already exists 
0d15b5ccf694: Pushed 
2a90fa3f4b5e: Layer already exists 
34f7184834b2: Layer already exists 
5836ece05bfd: Layer already exists 
72e830a4dff5: Layer already exists 
java-mysql-app: digest: sha256:1d68b5e2ff5825261d78cbaf1686118de4942f5988e12028ab0a889ae107878a size: 1576