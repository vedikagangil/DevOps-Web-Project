pipeline {
    agent any
    
    environment {
        // Application server details - UPDATE THESE FOR YOUR SETUP
        APP_SERVER_IP = 'YOUR_APP_SERVER_IP'  // Your Linux app server IP
        APP_USER = 'ec2-user'                 // Your Linux app server username
        DEPLOY_PATH = '/home/ec2-user/app'
        // Add Maven to PATH if needed
        PATH = "/opt/maven/bin:${env.PATH}"
    }
    
    tools {
        // Configure Maven in Jenkins globally first
        maven 'M3'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Unit Test') {
            steps {
                // Use sh for Linux commands if Jenkins is on Linux
                // If Jenkins is on Windows, ensure Maven is installed and in PATH
                bat 'mvn --version' // First check if Maven is available
                bat 'mvn clean compile test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                bat 'mvn package -DskipTests'
                archiveArtifacts 'target/*.war'
            }
        }
        
        stage('Deploy to Linux Server') {
            steps {
                script {
                    // For Windows Jenkins to Linux deployment
                    withCredentials([sshUserPrivateKey(credentialsId: 'your-ssh-credential-id', keyFileVariable: 'SSH_KEY')]) {
                        bat """
                            scp -o StrictHostKeyChecking=no -i "%SSH_KEY%" target/*.war ${env.APP_USER}@${env.APP_SERVER_IP}:${env.DEPLOY_PATH}/
                        """
                        
                        bat """
                            ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ${env.APP_USER}@${env.APP_SERVER_IP} "
                                cd ${env.DEPLOY_PATH}
                                # Stop existing app
                                sudo pkill -f 'java.*war' || true
                                sleep 5
                                # Start new app
                                nohup java -jar *.war > app.log 2>&1 &
                                echo 'Deployment completed!'
                            "
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Pipeline failed! Check the console output for details.'
        }
    }
}
