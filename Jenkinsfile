pipeline {
    agent any
    
    environment {
        // Application server details - UPDATE THESE FOR YOUR SETUP
        APP_SERVER_IP = 'YOUR_APP_SERVER_IP'  // Your Linux app server IP
        APP_USER = 'ec2-user'                 // Your Linux app server username
        DEPLOY_PATH = '/opt/your-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Unit Test') {
            steps {
                // Use bat for Windows commands
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
                    // Copy artifact to Linux application server
                    bat """
                        scp -o StrictHostKeyChecking=no -i "C:\\path\\to\\your\\key.pem" target/*.war ${env.APP_USER}@${env.APP_SERVER_IP}:${env.DEPLOY_PATH}/
                    """
                    
                    // SSH to Linux server and deploy
                    bat """
                        ssh -o StrictHostKeyChecking=no -i "C:\\path\\to\\your\\key.pem" ${env.APP_USER}@${env.APP_SERVER_IP} "
                            cd ${env.DEPLOY_PATH}
                            # Stop existing app
                            pkill -f 'java.*war' || true
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
