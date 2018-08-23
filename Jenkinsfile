@Library("jenkins-shared-library@master")_

// instantiate
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
                sh "apk update && apk add curl git"
                script {
                  Slack.populateGlobalVariables()
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm install react-scripts'
            }
        }
        stage('Test') {
            when {
                branch 'master'
            }
            steps {
                echo "Skip Test"
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deliver-Master') {
            when {
                branch 'master'
            }
            steps {
                sh './jenkins/scripts/deliver.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
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
                author_name: "${author}",
                text: "${currentBuild.currentResult}",
                "mrkdwn_in": ["fields"],
                fields: [
                    [
                        title: "Branch:",
                        value: "${env.GIT_BRANCH}",
                        short: true
                    ],
                    [
                        title: "Last Commit:",
                        value: "${message}",
                        short: false
                    ]
                ]
            ]
            ])
        }
   }
}
