pipeline {
    agent any
    
    tools {
        jdk 'java17'
        maven 'M3'
    }
    
    environment {
        DEPLOY_PATH = 'C:\\jenkins\\app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                bat 'mvn clean compile -DskipTests'
            }
        }
        
        stage('Package') {
            steps {
                bat 'mvn package -DskipTests'
                archiveArtifacts 'target/*.war'
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    bat """
                        echo "=== Deployment ==="
                        if not exist "${env.DEPLOY_PATH}" mkdir "${env.DEPLOY_PATH}"
                        copy target\\*.war "${env.DEPLOY_PATH}\\"
                        taskkill /F /IM java.exe 2>nul || echo "No running apps"
                        timeout /t 5
                        cd /d "${env.DEPLOY_PATH}"
                        for %%f in (*.war) do (
                            start "WebApp" /B java -jar "%%f" > app.log 2>&1
                        )
                        echo "Deployment completed!"
                    """
                }
            }
        }
        
        stage('Verify') {
            steps {
                script {
                    bat """
                        timeout /t 10
                        tasklist /FI "IMAGENAME eq java.exe"
                        curl -f http://localhost:8080/ && echo "SUCCESS!" || echo "App not responding"
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
