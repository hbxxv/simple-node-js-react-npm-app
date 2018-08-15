import groovy.json.JsonOutput
import java.util.Optional
import hudson.model.Actionable
import hudson.tasks.junit.CaseResult

def slackNotificationChannel = "spam"
def message = ""
def author = ""

def notifySlack(text, channel, attachments) {
    def slackURL = 'https://hooks.slack.com/services/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'
    def jenkinsIcon = 'https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png'

    def payload = JsonOutput.toJson([text: text,
        channel: channel,
        username: "Jenkins",
        icon_url: jenkinsIcon,
        attachments: attachments
    ])

    sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
}

def getGitAuthor = {
    def commit = sh(returnStdout: true, script: 'git rev-parse HEAD')
    author = sh(returnStdout: true, script: "git --no-pager show -s --format='%an' ${commit}").trim()
}

def getLastCommitMessage = {
    message = sh(returnStdout: true, script: 'git log --pretty=%s -1 | xargs echo').trim()
}

def populateGlobalVariables = {
    getLastCommitMessage()
    getGitAuthor()
}

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
                sh "apk update && apk add curl"
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            when {
                branch 'master'
            }
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deliver-Master') {
            when {
                branch 'master'
            }
            steps {
                sh './jenkins/scripts/deliver.sh'
                //input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }    
        }
    }
    post {
        always {
            echo 'I will always say Hello again!'
            notifySlack("", slackNotificationChannel, [
            [
                title: "${env.JOB_NAME}, build #${env.BUILD_NUMBER}",
                title_link: "${env.BUILD_URL}",
                color: "danger",
                author_name: "${env.GIT_COMMITTER_NAME}",
                text: "${currentBuild.currentResult}",
                "mrkdwn_in": ["fields"],
                fields: [
                    [
                        title: "Branch",
                        value: "${env.GIT_BRANCH}",
                        short: true
                    ],
                    [
                        title: "Last Commit",
                        value: "${message}",
                        short: false
                    ]
                ]
            ]
            ])
        }
   }
}
