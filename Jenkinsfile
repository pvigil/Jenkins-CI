#!groovy
import groovy.json.JsonOutput

def slackNotificationChannel = 'team-pge'     // ex: = "builds"

def notifySlack(text, channel, attachments) {
    def slackURL = 'https://modelituy.slack.com/services/hooks/jenkins-ci/'
    def jenkinsIcon = 'https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png'

    def payload = JsonOutput.toJson([text: text,
        channel: channel,
        username: "Jenkins",
        icon_url: jenkinsIcon,
        attachments: attachments
    ])

    sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
}

node {
    stage("Post to Slack") {
        notifySlack("Success!", slackNotificationChannel, [])
    }
}
