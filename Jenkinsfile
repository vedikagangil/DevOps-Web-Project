pipeline {
    agent any
    
    environment {
        // Application server details - UPDATE THESE FOR YOUR SETUP
        APP_SERVER_IP = 'YOUR_APP_SERVER_IP'  // Your target EC2 instance IP
        APP_USER = 'ec2-user'                 
        DEPLOY_PATH = '/home/ec2-user/app'
    }
    
    tools {
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
                sh 'mvn --version'
                sh 'mvn clean compile test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts 'target/*.war'
            }
        }
        
        stage('Deploy to App Server') {
            steps {
                script {
                    // Copy artifact to application server
                    sh """
                        scp -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/jenkins_deploy_key target/*.war ${env.APP_USER}@${env.APP_SERVER_IP}:${env.DEPLOY_PATH}/
                    """
                    
                    // SSH to application server and deploy
                    sh """
                        ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/jenkins_deploy_key ${env.APP_USER}@${env.APP_SERVER_IP} "
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
    
    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Pipeline failed! Check the console output for details.'
        }
    }
}
