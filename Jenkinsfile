pipeline {
    agent any
    
    environment {
        
        RENDER_API_TOKEN = credentials('Render')
        RENDER_APP_NAME = 'gallery'
        EMAIL_RECIPIENT = 'victormwanza30@gmail.com'
        SLACK_CHANNEL = '#victor_re'  
        SLACK_CREDENTIALS_ID = 'Slack'
    }
    
    tools {
        nodejs 'node-12'
    }
    
    stages {
        stage('Clone repository') {
            steps {
                echo 'Cloning repository...'
                git 'https://github.com/vicingit/gallery.git'
            }
        }
        
        stage('Install dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }

        stage('install mocha and chai'){
            steps{
                echo 'other dependencies...'
                sh 'npm install --save-dev mocha chai chai-http'
            }
        }
        stage('Test project'){
            steps{
                echo 'Testing...'
                sh 'npm test'
            }
        }

        stage('Build project') {
            steps {
                echo 'Building...'
                sh 'npm run build'
            }
        }
        
        stage('start server'){
            steps{
                echo 'starting server...'
                sh 'npm start &'
                sleep 10 //time for the server to start

                
            }
        }
        
        stage('Deploy to Render') {
            steps {
                echo 'Deploying to Render...'
                script {
                    // Use curl to trigger a deployment to Render
                    sh "curl -X POST -H 'Authorization: Bearer ${RENDER_API_TOKEN}' \
                        -H 'Content-Type: application/json' \
                        -d '{\"branch\": \"master\", \"env\": {\"NODE_ENV\": \"production\"}}' \
                        https://api.render.com/v1/service/${RENDER_APP_NAME}/deploy"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment to Render succeeded!'
            echo 'SlackBot success message'
            slackSend channel: "${SLACK_CHANNEL}", color: 'good', message: "Build succeeded: ${env.JOB_NAME} ${env.BUILD_NUMBER}. Access app on https://week2ip1.onrender.com/"
        }
        failure {
            echo 'Deployment to Render failed!'
            echo 'SlackBot failed message'
            slackSend channel: "${SLACK_CHANNEL}", color: 'danger', message: "Build failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        always{
            script{
                

                if(currentBuild.result == 'FAILURE'){
                   emailext (
                        to: "${EMAIL_RECIPIENT}",
                        subject: "Jenkins Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                        body: """
                        <p>The Jenkins build <b>${env.JOB_NAME} ${env.BUILD_NUMBER}</b> has failed.</p>
                        <p>Please check the Jenkins console output for more details: ${env.BUILD_URL}</p>
                        """
                    ) 
                }
            }
        }
    }
}