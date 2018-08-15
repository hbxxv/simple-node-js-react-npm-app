def call(String buildResult) {
  if ( buildResult == "SUCCESS" ) {
    slackSend color: "good", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was successful"
  }
  else if( buildResult == "FAILURE" ) { 
    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was failed"
  }
  else if( buildResult == "UNSTABLE" ) { 
    slackSend color: "warning", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was unstable"
  }
  else {
    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} its resulat was unclear"	
  }
}

pipeline {
    agent {
        docker {
            image 'node:6-alpine' 
            args '-p 3000:3000' 
        }
    }
    environment {
            CI = true
            REVISION_ID = "${env.BUILD_NUMBER}-${env.GIT_COMMIT}"
        }
    stages {
        stage('Build & Test') {
            parallel {
                stage('Build') { 
                    steps {
                        sh 'npm install'
                    }
                }
                stage('Test') {
                    steps {
                        sh './jenkins/scripts/test.sh'
                    }
                }
		stage('Print Info') {
		   steps {
			echo "${REVISION_ID}"
			echo "${env.REVISION_ID}"
			echo "${env.BUILD_NUMBER}"
			echo "${env.GIT_COMMIT}"
		   }
		}
            }
        }
        stage('Deliver-DEV') {
            when {
                branch 'DEV'
            }
            steps {
                sh './jenkins/scripts/deliver.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
            
        }
        stage('Deliver-Master') {
            when {
                branch 'master'
            }
            steps {
                //script {
                    //env['GIT_MSG'] = sh(script: "git log --pretty=%s -1", returnStdout: true).trim()
                    //env['GIT_MSG'] = sh 'git log --pretty=%s -1 | xargs echo'
                    //env['GIT_COMMIT'] = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    //env['GIT_AUTHOR'] = sh(script: "git --no-pager show -s --format='%an' \${GIT_COMMIT}", returnStdout: true).trim()
                //}
                sh './jenkins/scripts/deliver.sh'
                //input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
                //echo "The commit \${GIT_COMMIT} was authored by \${GIT_AUTHOR}. Commit message was \"\${GIT_MSG}\""
            }    
        }
	post {
            always {
	       /* Use slackNotifier.groovy from shared library and provide current build result as parameter */   
                slackNotifier(currentBuild.currentResult)
                cleanWs()
            }
        }
    }

}
