def slackNotifier(String buildResult) {
  if ( buildResult == "SUCCESS" ) {
    slackSend color: "good", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} committed by ${env.GIT_COMMITTER_NAME} was successful, link ${env.BUILD_URL}"
  }
  else if( buildResult == "FAILURE" ) { 
    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} committed by ${env.GIT_COMMITTER_NAME} was failed, link ${env.BUILD_URL}"
  }
  else if( buildResult == "UNSTABLE" ) { 
    slackSend color: "warning", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} committed by ${env.GIT_COMMITTER_NAME} was unstable, link ${env.BUILD_URL}"
  }
  else {
    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} committed by ${env.GIT_COMMITTER_NAME} its resulat was unclear, link ${env.BUILD_URL}"
  }
}

import groovy.json.JsonOutput


def notifySlack(text, channel, attachments) {
    def slackURL = 'https://hooks.slack.com/services/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'
    def jenkinsIcon = 'https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png'

    def payload = JsonOutput.toJson([text: text,
        channel: spam,
        username: "Jenkins",
        icon_url: jenkinsIcon,
        attachments: attachments
    ])

    sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
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
    }
    post {
        always {
            /* Use slackNotifier.groovy from shared library and provide current build result as parameter */
            echo 'I will always say Hello again!'
            //slackNotifier(currentBuild.currentResult)
            notifySlack("", slackNotificationChannel, [
                [
                    title: "${env.JOB_NAME}, build #${env.BUILD_NUMBER}",
                    title_link: "${env.BUILD_URL}",
                    color: "danger",
                    author_name: "${env.GIT_COMMITTER_NAME}",
                    text: "${env.buildResult}",
                    fields: [
                        [
                            title: "Branch",
                            value: "${env.GIT_BRANCH}",
                            short: true
                        ],
                        [
                            title: "Last Commit",
                            value: "TEST",
                            short: false
                        ]
                    ]
                ]
            ]
            //cleanWs()
        }
   }
}
