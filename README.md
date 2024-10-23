# Casestudy
AdvDevopsCaseStudy



case Study Overview
The case study involves integrating static analysis tools with Infrastructure-as-Code (IaC) using Terraform. This project focuses on setting up a development environment that combines Jenkins for continuous integration and deployment, SonarQube for static code analysis, and Terraform for infrastructure provisioning. The goal is to create a robust pipeline that can automatically analyze Python applications for quality and security issues.

Key Features and Applications:
Key features of this case study include:

Infrastructure deployment using Terraform
Continuous Integration/Continuous Deployment (CI/CD) pipeline setup
Static code analysis integration
Cross-tool communication between Jenkins, SonarQube, and Terraform The practical applications of this setup are:
Improved code quality through automated static analysis
Enhanced security by identifying potential vulnerabilities early in the development cycle
Streamlined development workflow with automated testing and reporting
Scalable infrastructure management using Terraform
Step-by-Step Explanation
Terraform
Step 1: Install terraform and add it to environment variable. Now, download Amazon CLI by visiting the following website. Visit https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Now, click on install https://awscli.amazonaws.com/AWSCLIV2.msi Complete the installation process for AWSCLIV2

Step 2: Open AWS Academy and now click on AWS Details and then click on show button present in front of AWS CLI label. You will be shown with your cedentials

Step 3: Now, create a folder in VSCode and create a main.tf file in it with the following content.

# Specify the AWS provider
provider "aws" {
  region = "us-east-1"  # Replace with your preferred region
}
# Jenkins instance
resource "aws_instance" "jenkins" {
  ami           = "ami-05f408238af346b4f"  # Amazon Linux 2 AMI
  instance_type = "t2.micro"
  key_name      = "AMjenkins"
  tags = {
    Name = "JenkinsServer"
  }
  # User data to install Jenkins with Java 17
  user_data = <<-EOF
  #!/bin/bash
  sudo yum update -y
  sudo dnf install -y java-17-amazon-corretto-devel  # Install Java 17
  sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  sudo yum install -y Jenkins
  sudo systemctl start Jenkins
  sudo systemctl enable Jenkins
  EOF
}
# SonarQube instance
resource "aws_instance" "sonarqube" {
  ami           = "ami-05f408238af346b4f"  # Amazon Linux 2 AMI
  instance_type = "t2.medium"
  key_name      = "AMsonarqube"
  tags = {
    Name = "SonarQubeServer"
  }
  # User data to install SonarQube manually
  user_data = <<-EOF
  #!/bin/bash
  sudo yum update -y
  sudo su -
  cd /opt
  wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.7.0.96327.zip
  unzip sonarqube-10.7.0.96327.zip
  sudo adduser sonar
  sudo passwd sonar
  sudo chown -R sonar:sonar /opt/sonarqube-10.7.0.96327
  su - sonar -c "/opt/sonarqube-10.7.0.96327/bin/linux-x86-64/sonar.sh start"
  EOF
}

Step 4: Now, to ensure and run the aws cli commands in vs code terminal, run the following commands

aws –version
aws configure
(Write the content as mentioned in the figure below) Step 5: Now, run the following commands in the vs code terminal to set the credential secrets.

$env:AWS_ACCESS_KEY_ID="<YOUR_AWS_ACCESS_KEY_ID>"
$env:AWS_SECRET_ACCESS_KEY="<YOUR_AWS_SECRET_ACCESS_KEY>"
$env:AWS_SESSION_TOKEN="<YOUR_AWS_SESSION_TOKEN>"
Step 6: Now, to check whether the aws cli is connected to your aws account run the following command.

aws sts get-caller-identity
aws configure
Step 7: Now, to get the AMI ID run the following command and select any of the AMI ID and replace the AMI ID present in main.tf file in VSCode.

aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-2.0.*-x86_64-gp2" --query "Images[*].[ImageId,Name]" --region us-east-1 --output table
After this run the following command

terraform init 
terraform apply
Step 8: After this Terraform will automatically create 2 EC2 instances on the EC2 Dashboard. To check the running instances: Visit AWS EC2 Dashboard.

Set up Security Groups for the given two instances
Step 1: Go to EC2 Dashboard and select the Security Groups present in the left pane or sidebar and the click on the create security group.

Create a security group with name AMjenkins-security and give some description and add the inbounds rules given below.
Create a security group with name AMsonarqube-security and give some description and add the inbounds rules given below.
Installation for Jenkins
Reference Video: https://www.youtube.com/watch?v=bNuAS52ebLs

Step 1: Click on the JenkinsServer and click on connect.

Step 2: Open Git Bash and go to the directory which has the Key downloaded. If you don’t have the key downloaded, create a key pair and download the .pem file for the key. Since, I have the key downloaded in Downloads directory, I used the following commands: cd Download dir AMjenkins.pem* ssh -i "AMjenkins.pem" ec2-user@ec2-98-80-223-40.compute-1.amazonaws.com

Step 3: Go to google and search for Jenkins and then click on the Download and Deploy Link. Else, navigate using the following link: https://pkg.jenkins.io/redhat-stable/

Step 4: Now, run the initial 2 commands as it is and then run the next 2 commands using sudo word in the first; to run as root user.

OR Run the following commands:

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install fontconfig java-17-openjdk
sudo yum install jenkins
Step 5: Now, in order to install java run the following commands:

sudo yum install java-17-amazon-corretto-headless
sudo yum install java-17-amazon-corretto
sudo dnf install java-17-amazon-corretto-devel
Step 6: Run the following commands:

sudo yum install Jenkins
sudo systemctl status jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
Step 7: Now, go to EC2 dashboard and select Jenkins server and copy its public address and visit http://:8080

Step 8: You will be redirected to this page on successful installation of Jenkins and visiting the public address url with port 8080.

Step 9: Now, come back to gitbash run the command

sudo more /var/lib/jenkins/secrets/initialAdminPassword
And, copy the content in the output and paste it in the input of Administrator password.

Step 10: Select install suggested plugins and complete the installation and initial configurations.

Password: Anuprita@4321

Step 11: After proper initial configuration you will be redirected to this page.

Step 12: Install more plugins which will be required for this experiment. a. SonarQube Scanner b. Pipeline: Stage View

Step 13: Go to Manage Jenkins > Tools. Scroll down to SonarQube Scanner installations and add the SonarQube Scanner and then click on the save button.

Sonarqube installation
Reference video: https://www.youtube.com/watch?v=E5hMOGeBT-o&t=38s

Step 1: Click on the SonarQubeServer and click on connect.

Step 2: Open Git Bash and go to the directory which has the Key downloaded. If you don’t have the key downloaded, create a key pair and download the .pem file for the key. Since, I have the key downloaded in Downloads directory, I used the following commands: cd Download dir SCsonarqube.pem* ssh -i "SCsonarqube.pem" ec2-user@ec2-98-80-223-40.compute-1.amazonaws.com

Step 3: Now, in order to install java run the following commands:

sudo su
sudo yum install java-17-amazon-corretto-headless
sudo yum install java-17-amazon-corretto
sudo dnf install java-17-amazon-corretto-devel
Step 4: Now, run the following command to install sonarqube:

sudo wget -O /etc/yum.repos.d/sonar.repo http://downloads.sourceforge.net/project/sonar-pkg/rpm/sonar.repo
sudo yum install sonar -y
Step 5: Now, run the following commands to install the sonarqube

sudo su
cd /opt
ll
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.7.0.96327.zip
unzip sonarqube-10.7.0.96327.zip
ll
cd sonarqube-10.7.0.96327
ll
cd conf
ll
cat sonar.properties
cd ..
cd bin
cd linux-x86-64
ll
./sonar.sh start
sudo adduser sonar
sudo passwd sonar
sudo chown -R sonar:sonar /opt/sonarqube-10.7.0.96327
su – sonar
cd /opt/sonarqube-10.7.0.96327/bin/linux-x86-64/
./sonar.sh start
./sonar.sh status
Step 6: Now, go to EC2 dashboard and select SonarQubeServer and copy its public address and visit http://:9000

Step 7: You will be redirected to this page on successful installation of SonarQube and visiting the public address url with port 9000. Login the username=admin and password=admin.

Step 8: Now, set up the initial configurations by setting up new password. password=Sonamsonam@2004

Step 9: Now, click on the Create a local project link and name the project Hello-World and choose use the global setting

Step 10: Click on Administration > Security > Users. And then give a token name and click on the generate token button and copy the token number and save it somewhere. For now, I have saved it in notepad.

GitHub
Step 1: Make a repository and upload you files in the repository.

Pipeline
Step 1: Open the git bash for Jenkins and run the following commands in the terminal to install git.

sudo yum install git
git –version
Step 2: Go to Manage Jenkins > System. Scroll down to SonarQube Servers section and name it as SonarQube Server and copy the http://:9000 Also, copy the token as secret here in secret text.

Step 3: Go to Manage Jenkins > Credentials. Copy this id and you will need to paste in the Pipeline Script later.

Step 4: Create a new pipeline and name it pipeline1.

Step 5: Now select on the Git project and paste your GitHub url.

Step 6: Now, write the following Pipeline Script.

pipeline {
  agent any
  stages {
    stage('Clone Repository') {
      steps {
        git branch: 'main', url: 'Add your url'
      }
    }
    stage('SonarQube Analysis') {
      environment {
        scannerHome = tool 'SonarQubeScanner'  // Ensure SonarQube Scanner is installed
      }
      steps {
        withSonarQubeEnv('SonarQube') {  // Name of SonarQube server configured in Jenkins
          withCredentials([string(credentialsId: '6e0ad648-6931-48d0-a2eb-938a55db6234', variable: 'SONAR_TOKEN')]) {
            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Hello-World -Dsonar.sources=. -Dsonar.login=$SONAR_TOKEN"
          }
        }
      }
    }
  }
  post {
    always {
      echo 'Pipeline completed'
    }
  }
}
Step 7: Build run the pipeline. It gives success. Also, check the console.

SonarQube Analysis and Results
Step 1: Visit back to the http://:9000 Now, go to projects section and you can see the analysis of the python project.

About
No description, website, or topics provided.
Resources
 Readme
 Activity
Stars
 0 stars
Watchers
 1 watching
Forks
 0 forks
Report repository
Releases
No releases published
Packages
No packages published
Footer
© 2024 GitHub, Inc.
Footer navigation
Term
