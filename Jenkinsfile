@Library("jenkins-shared-library@master")_

pipeline {
    agent {
        docker {
            image 'node:6-alpine' 
            args '-p 3000:3000' 
        }
    }
    environment {
            CI = 'true'
            REVISION_ID = "${env.BUILD_NUMBER}-${env.GIT_COMMIT}"
        }
    stages {
        stage('Intiliaze') {
            steps {
                //sh "apk update && apk add curl git"
                echo "Skip"
            }
        }
        stage('Build') {
            steps {
                //sh 'npm install'
                //sh 'npm install react-scripts'
            }
        }
        stage('Test') {
            when {
                branch 'master'
            }
            steps {
                echo "Skip Test"
                //sh './jenkins/scripts/test.sh'

            }
        }
        stage('Deliver-Master') {
            when {
                branch 'master'
            }
            steps {
                echo "Skip"
                //sh './jenkins/scripts/deliver.sh'
                //input message: 'Finished using the web site? (Click "Proceed" to continue)'
                //sh './jenkins/scripts/kill.sh'
            }    
        }
    }
    post {
        always {
            echo 'I will always say Hello again!'
            Slack()
        }
   }
}
