pipeline {
    agent any
    
    tools {
        maven 'M3'  // This references the Maven installation you just configured
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
                bat 'mvn --version'
                bat 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                bat 'mvn test'
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
        
        stage('Deploy') {
            steps {
                script {
                    bat """
                        echo "=== Starting Deployment ==="
                        if not exist "${env.DEPLOY_PATH}" mkdir "${env.DEPLOY_PATH}"
                        copy target\\*.war "${env.DEPLOY_PATH}\\"
                        taskkill /F /IM java.exe 2>nul || echo "No existing app"
                        timeout /t 5
                        cd /d "${env.DEPLOY_PATH}"
                        for %%f in (*.war) do (
                            start /B java -jar "%%f" > app.log 2>&1
                        )
                        echo "Deployment completed!"
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
