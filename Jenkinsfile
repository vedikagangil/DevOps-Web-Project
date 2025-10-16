pipeline {
    agent any
    
    environment {
        DEPLOY_PATH = '/home/ec2-user/app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn --version'
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
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
        
        stage('Deploy') {
            steps {
                script {
                    sh """
                        echo "=== Starting Deployment ==="
                        
                        # Create deployment directory
                        echo "Creating deployment directory at ${env.DEPLOY_PATH}"
                        sudo mkdir -p ${env.DEPLOY_PATH}
                        sudo chown \$USER:\$USER ${env.DEPLOY_PATH}
                        
                        # Copy WAR file
                        echo "Copying WAR file to deployment directory"
                        cp -v target/*.war ${env.DEPLOY_PATH}/
                        
                        # Stop any existing application
                        echo "Stopping any existing application..."
                        sudo pkill -f 'java.*war' || echo "No existing application running"
                        sleep 5
                        
                        # Deploy new application
                        echo "Starting new application..."
                        cd ${env.DEPLOY_PATH}
                        nohup java -jar *.war > app.log 2>&1 &
                        echo \$! > app.pid
                        
                        echo "Application started with PID: \$(cat app.pid)"
                        echo "Deployment completed successfully!"
                    """
                }
            }
        }
        
        stage('Verify') {
            steps {
                script {
                    sh """
                        echo "=== Verifying Deployment ==="
                        
                        # Wait for app to start
                        sleep 10
                        
                        # Check if process is running
                        echo "Checking application process..."
                        if ps -p \$(cat ${env.DEPLOY_PATH}/app.pid) > /dev/null 2>&1; then
                            echo "✓ Application is running with PID: \$(cat ${env.DEPLOY_PATH}/app.pid)"
                        else
                            echo "✗ Application process not found"
                            echo "Application log:"
                            cat ${env.DEPLOY_PATH}/app.log
                            exit 1
                        fi
                        
                        # Check listening ports
                        echo "Checking listening ports..."
                        netstat -tlnp | grep :8080 || echo "Port 8080 not in use (this might be normal)"
                        
                        # Try to access the application
                        echo "Testing application endpoint..."
                        curl -f http://localhost:8080/ && echo "✓ Application is accessible" || echo "⚠ Application not accessible on port 8080 (might be using different port)"
                        
                        # Show application log
                        echo "Application log (last 10 lines):"
                        tail -10 ${env.DEPLOY_PATH}/app.log
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "=== Pipeline Complete ==="
            cleanWs()
        }
        success {
            echo "🎉 Pipeline completed successfully!"
            sh """
                echo "Application deployed to: ${env.DEPLOY_PATH}"
                echo "Log file: ${env.DEPLOY_PATH}/app.log"
                echo "PID file: ${env.DEPLOY_PATH}/app.pid"
            """
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above for details."
        }
    }
}
