pipeline {
    agent {
        docker {
            image 'node:6-alpine' 
            args '-p 3000:3000' 
        }
    }
    environment {
            CI = true
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
			sh 'echo $BUILD_NUMBER-$GIT_COMMIT'
			sh 'export REVISION_ID=$BUILD_NUMBER-$GIT_COMMIT'
			sh 'echo $REVISION_ID'	
		   }
		}
            }
        }
        stage('Sonar-Anal') {
            when {
                anyOf { branch 'master'; branch 'DEV'; branch 'Staging'}
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
    }

}
