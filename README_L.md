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