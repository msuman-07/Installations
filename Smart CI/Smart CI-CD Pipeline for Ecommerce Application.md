First Create 2 instances (SonarQube & Jenkins) (All Traffic)

In SonarQube-ec2 installations - 

sudo apt update 
sudo apt upgrade -y 
sudo apt install -y openjdk-21-jdk 
java -version

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" /etc/apt/sources.list.d/pgdg.list' 

sudo wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add - 

sudo apt install postgresql postgresql-contrib -y 

sudo systemctl enable postgresql 
sudo systemctl start postgresql 
sudo systemctl status postgresql 

psql --version 

sudo -i -u postgres 

createuser ddsonar 

psql 

ALTER USER ddsonar WITH ENCRYPTED password 'user'; 
CREATE DATABASE ddsonarqube OWNER ddsonar; 
GRANT ALL PRIVILEGES ON DATABASE ddsonarqube to ddsonar; 

\l
\du
\q
exit 

sudo apt install zip -y 
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-26.2.0.119303.zip 
sudo unzip sonarqube-26.2.0.119303.zip 
sudo mv sonarqube-26.2.0.119303 sonarqube 
sudo mv sonarqube /opt/ 
sudo groupadd ddsonar 
sudo useradd -d /opt/sonarqube -g ddsonar ddsonar 
sudo chown ddsonar:ddsonar /opt/sonarqube -R 

sudo nano /opt/sonarqube/conf/sonar.properties 

#under DATABASE user credentials 

sonar.jdbc.username=ddsonar 
sonar.jdbc.password=user 
sonar.jdbc.url=jdbc:postgresql://localhost:5432/ddsonarqube 

(Remove all # for these 3 commands)
Then click "Ctrl + x" , "y" , "Enter" 


sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh

(type the below command at the end of the nano)
RUN_AS_USER=ddsonar 

Then click "Ctrl + x" , "y" , "Enter"

sudo nano /etc/systemd/system/sonar.service 

Copy and paste the below commands inside it - 


[Unit] 
Description=SonarQube service 
After=syslog.target network.target
[Service] 
Type=forking 
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start 
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop 
User=ddsonar 
Group=ddsonar 
Restart=always 
LimitNOFILE=65536 
LimitNPROC=4096 
[Install] 
WantedBy=multi-user.target


Then click "Ctrl + x" , "y" , "Enter"

sudo systemctl enable sonar
sudo systemctl start sonar 
sudo systemctl status sonar 

###(If error only then change java version and restart or else continue below)  
sudo apt install -y openjdk-21-jdk 
sudo systemctl restart sonar

Continue -

sudo nano /etc/sysctl.conf

(Copy & Paste the below commands at the end of the nano)

vm.max_map_count=262144 
fs.file-max=65536
ulimit -n 65536 
ulimit -u 4096

Then click "Ctrl + x" , "y" , "Enter"

• Reboot System
sudo reboot

Open new tab -> Pasted IP Address in new tab -> 9000 is port number of SonarQube (:9000)

In Sonar Qube Webpage ->
Username - admin
Password - admin


Inside sonarqube webpage - 

Quality Gates -> Create -> Create Quality Gate Name - gate -> Create 

Delete -> Security Hotspots Reviewed

Add Condition -> On New Code -> Qlty Gate fails when -> Block -> In Drop down Type - "Block Severity Issues" -> Opr - is greater than -> value - 0

These conditions should be there 

Issues - is greater than - 0
Coverage - is less than - 70.0%
Duplicated Lines - is greater than - 3.0% 
Block Severity Issues - is greater than - 0

Homepage -> Click Administration -> Security -> Users -> Click three dotes below Tokens (To generate token) -> Enter token name as "sonar-token" -> Generate -> Copy and save the below Code on notepad

Now jenkins-ec2(server)

sudo su

sudo apt update -y 

sudo apt install -y fontconfig openjdk-21-jre 

sudo apt install maven -y 

sudo mkdir -p /etc/apt/keyrings

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update 

sudo apt install -y jenkins

sudo systemctl status jenkins

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

In Jenkins Webpage -> Setting -> Plugins -> Search "SonarQube Scanner" -> Install it

Install Plugins - 
SonarQube Scanner for Jenkins
Email Extension Plugin
Maven Integration Plugin

Settings -> Tools -> + Add Maven Installations -> Name - Maven -> Install Automatically -> Apply and Save it

Settings -> Credentials -> Global -> Add Credentials -> Click Secret Text ->  Scope - Global (Jenkins, nodes.....) -> Secret - Add the sonarqube token we copied on notepad earlier -> ID - SonarQube -> Description - SonarQube -> Create

Settings -> System -> SonarQube Installations -> Name - SonarQube (Add secret Text Id as a name) -> Server URL - Add Sonar Server EC2 public id with port number 9000 -> Server auth token - In Dropdown click SonarQube

Jenkins -> New Item -> Item Name - EcommerceApp -> pipeline -> ok 

General -> Description - This is Ecommerce Project -> Click "Github Project" -> Project URL - https://github.com/Msocial123/EcommerceApp.git  

Triggers -> Poll SCM -> Schedule - "....." (Use five dots only) 

Pipeline script -> Add the below pipeline code -> Cick on Apply and Save 

Pipeline script -
-----------------------------------------------------------------------------------------------

pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Msocial123/EcommerceApp.git'
            }
        }

        stage('Check Project Structure') {
            steps {
                sh 'ls -la'
            }
        }

        stage('Compile') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Test') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn test'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('EcommerceApp') {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Package') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn package'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                dir('EcommerceApp') {
                    sh 'sudo cp target/EcommerceApp.war ${TOMCAT_PATH}'
                }
            }
        }
    }

    post {

        success {
            emailext(
                to: 'your-email@gmail.com',
                subject: "SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
Build succeeded ✅

Job Name: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}

The application has been successfully built and deployed.
"""
            )
        }

        failure {
            emailext(
                to: 'your-email@gmail.com',
                subject: "FAILED: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
Build failed ❌

Job Name: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}

Please check the console output for error details.
"""
            )
        }
    }
}

-----------------------------------------------------------------------------------------------

"Click Apply to save in Jenkins"

Email Configuration -

Setting -> Click on Manage Jenkins -> System -> E-mail Notification -> SMTP Server - smtp.gmail.com -> Default - @gmail.com -> Advanced -> Use SMTP Authentication -> User Name - msuman2003711@gmail.com (use your email-id) -> For Password - Need Google App password

Go to Your Mail-ID -> Manage your Google Account -> Security & Sign-in -> Search - App Passwords -> Also Turn on 2-step verification -> App Name - Jenkins -> Copy the Generated app password without spaces

Now add that saved password key on Jenkins without space -> Tick - Use SSL -> SMTP Port - 465 -> Charset - UTF-8 -> Tick - Test configuration by sending test e-mail - "msuman2003711@gmail.com" (your mail-id)

Scroll up and Find -> Extended E-Mail Notification -> SMTP server - smtp.gmail.com -> SMTP Port - 465 -> Advanced -> Add - Use OAuthen 2.0 -> Username with password ->

Scope - Global(Jenkins,nod...) -> Username - msuman2003711@gmail.com -> pw - Add google pass key which you have generated -> ID - Email -> Descp - Email -> Create

Click on checkbox - Use SSL -> Below -> Default user e-mail suffix - @gmail.com -> Default Content Type - HTML -> Apply and save it

Check both E-Mail Notification & Extended E-Mail Notification is everything filled

### Apache Tomcat Installation & Setup ###

Go to Ubuntu user home
(Don’t stay in /root)

cd /home/ubuntu

Check:
pwd

Output must be:
/home/ubuntu

Downloads Tomcat server - 
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.89/bin/apache-tomcat-9.0.89.tar.gz
(Get the tomcat installation zip from that website - https://tomcat.apache.org/download-90.cgi)

Extracts compressed file - 
tar -xvzf apache-tomcat-9.0.89.tar.gz

You will get -
apache-tomcat-9.0.89

Rename Folder -
mv apache-tomcat-9.0.89 tomcat9

Now your folder -
/home/ubuntu/tomcat9

Give Permission - 
sudo chown -R ubuntu:ubuntu /home/ubuntu/tomcat9

Go to Config Folder - 
cd tomcat9/conf

Change Tomcat Port -
Open config -

vi server.xml

Find -
Connector port="8080"

Change 8080 -> 9090

Save and Exit Vim -

Click "i" for insert
Esc
Type :wq
Enter

Go to bin folder - 
cd ../bin

Give Execute Permission - 
chmod +x *.sh

Start Tomcat - 
./startup.sh

Jenkins Permission Setup -

sudo visudo

Go to bottom and add this command -
jenkins ALL=(ALL) NOPASSWD: ALL

Then click "Ctrl + x" , "y" , "Enter"

" Setup Complete "


Access Tomcat (Use Jenkins server ip)
http://"server-ip":9090

Go to Jenkins -> Correct the pipeline code with "your-mail-id" and "SonarQube" name once in Jenkins -> Click BUILD NOW -> Click Pipeline Overview -> Everything must be tick like this:

Start -> Tool Install -> Clone Repository -> Check Project Structure -> Compile -> Test -> SonarQube Analysis -> Package -> Deploy to Tomcat -> Download Sonar... -> Post Actions -> End

Correct and add these in the above pipeline code if needed - 


stage('Download Sonar Report') {
    steps {
        sh """
            curl -u ${SONAR_TOKEN}: \
            http://<SONARQUBE_PUBLIC_IP>:9000/api/issues/search?projectKeys=${SONAR_PROJECT_KEY} \
            -o sonar-report.json
        """
    }
}

stage('Push Report to GitHub') {
    steps {
        sh """
            git config user.email "jenkins@ci.com"
            git config user.name "Jenkins"

            git add sonar-report.json
            git commit -m "SonarQube Report - Build ${BUILD_NUMBER}" || echo "No changes to commit"

            git push origin master
        """
    }
}


In SonarQube Webpage the "EcommerceApp Maven WebApp" must RUN

Check your Mail for the confirmation process like "✔️ SUCCESS: EcommerceApp - Build #1" or "❌ FAILED: EcommerceApp - Build #1"

Click BUILD NOW after every CONFIGURATION

#############################################       MY PIPELINE SCRIPT      ################################################


pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        SONAR_TOKEN = "squ_4503a72ed1ba69cc36ebc87b06e05ddd4fac4243"
        SONAR_PROJECT_KEY = "EcommerceApp"
        SONAR_HOST_URL = "http://13.202.45.152:9000"
        TOMCAT_PATH = "/home/ubuntu/tomcat9/webapps/"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Msocial123/EcommerceApp.git'
            }
        }

        stage('Check Project Structure') {
            steps {
                sh 'ls -la'
            }
        }

        stage('Compile') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Test') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn test'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('EcommerceApp') {
                    withSonarQubeEnv('SonarQube') {
                        sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Package') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn package'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                dir('EcommerceApp') {
                    sh "sudo cp target/EcommerceApp.war ${TOMCAT_PATH}"
                }
            }
        }

        stage('Download Sonar Report') {
            steps {
                sh """
                curl -u ${SONAR_TOKEN}: \
                ${SONAR_HOST_URL}/api/issues/search?projectKeys=${SONAR_PROJECT_KEY} \
                -o sonar-report.json
                """
            }
        }
    }

    post {

        success {
            emailext(
                to: 'msuman2003711@gmail.com',
                subject: "✔️ SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
<h2 style="color:green;">✔️ Build Successful</h2>

<b>Job:</b> ${env.JOB_NAME}<br>
<b>Build Number:</b> ${env.BUILD_NUMBER}<br>
<b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a><br><br>

Application built and deployed successfully.
"""
            )
        }

        failure {
            emailext(
                to: 'msuman2003711@gmail.com',
                subject: "❌ FAILED: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
<h2 style="color:red;">❌ Build Failed</h2>

<b>Job:</b> ${env.JOB_NAME}<br>
<b>Build Number:</b> ${env.BUILD_NUMBER}<br>
<b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a><br><br>

Check Jenkins console for errors.
"""
            )
        }
    }
}

#############################################       MY PIPELINE SCRIPT      ################################################

