#!groovy

import groovy.json.JsonOutput
import java.util.Optional
import hudson.tasks.test.AbstractTestResultAction
import hudson.model.Actionable
import hudson.tasks.junit.CaseResult

def speedUp = '--configure-on-demand --daemon --parallel'
def nebulaReleaseScope = (env.BRANCH_NAME == 'origin/master') ? '' : "-Prelease.scope=patch"
def nebulaRelease = "-x prepare -x release snapshot ${nebulaReleaseScope}"
def gradleDefaultSwitches = "${speedUp} ${nebulaRelease}"
def gradleAdditionalTestTargets = "integrationTest"
def gradleAdditionalSwitches = "shadowJar"
def slackNotificationChannel = "team-pge"
def author = ""
def message = ""
def testSummary = ""
def total = 0
def failed = 0
def skipped = 0

echo "Branch Name: ${env.BRANCH_NAME}"
echo "Change Id: ${env.CHANGE_ID}"
echo "Change Url: ${env.CHANGE_URL}"
echo "Change Title: ${env.CHANGE_TITLE}"
echo "Change Author: ${env.CHANGE_AUTHOR}"
echo "Change Author Display Name: ${env.CHANGE_AUTHOR_DISPLAY_NAME}"
echo "Change Target: ${env.CHANGE_TARGET}"


def isPublishingBranch = { ->
    return env.BRANCH_NAME == 'origin/master' || env.BRANCH_NAME =~ /release.+/
}

def isResultGoodForPublishing = { ->
    return currentBuild.result == null
}

def notifySlack(text, channel, attachments) {
    def slackURL = 'https://modelituy.slack.com/services/hooks/jenkins-ci'
    def jenkinsIcon = 'https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png'

    def payload = JsonOutput.toJson([text: text,
        channel: channel,
        username: "Jenkins",
        icon_url: jenkinsIcon,
        attachments: attachments
    ])

    slackSend(color: '#00FF00', channel: 'team-pge', attachments: JsonOutput.toJson(attachments))
    //sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
}

def getGitAuthor = {
    def commit = sh(returnStdout: true, script: 'git rev-parse HEAD')
    author = sh(returnStdout: true, script: "git --no-pager show -s --format='%an' ${commit}").trim()
}

def getLastCommitMessage = {
    message = sh(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
}

@NonCPS
def getTestSummary = { ->
    def testResultAction = currentBuild.rawBuild.getAction(AbstractTestResultAction.class)
    def summary = ""

    if (testResultAction != null) {
        total = testResultAction.getTotalCount()
        failed = testResultAction.getFailCount()
        skipped = testResultAction.getSkipCount()

        summary = "Passed: " + (total - failed - skipped)
        summary = summary + (", Failed: " + failed)
        summary = summary + (", Skipped: " + skipped)
    } else {
        summary = "No tests found"
    }
    return summary
}

@NonCPS
def getFailedTests = { ->
    def testResultAction = currentBuild.rawBuild.getAction(AbstractTestResultAction.class)
    def failedTestsString = "```"

    if (testResultAction != null) {
        def failedTests = testResultAction.getFailedTests()
        
        if (failedTests.size() > 9) {
            failedTests = failedTests.subList(0, 8)
        }

        for(CaseResult cr : failedTests) {
            failedTestsString = failedTestsString + "${cr.getFullDisplayName()}:\n${cr.getErrorDetails()}\n\n"
        }
        failedTestsString = failedTestsString + "```"
    }
    return failedTestsString
}

def populateGlobalVariables = {
    getLastCommitMessage()
    getGitAuthor()
    testSummary = getTestSummary()
}

node {
    try {
        stage('Checkout') {
            //checkout scm
            def scmVars = checkout scm
            def commitHash = scmVars.GIT_COMMIT

            echo "Commit Hash: ${commitHash}"
        }

        stage('Build') {
            //sh "./gradlew ${gradleDefaultSwitches} clean build ${gradleAdditionalTestTargets} ${gradleAdditionalSwitches} --refresh-dependencies"
            //step $class: 'JUnitResultArchiver', testResults: '**/TEST-*.xml'

            populateGlobalVariables()

            def buildColor = currentBuild.result == null ? "good" : "warning"
            def buildStatus = currentBuild.result == null ? "Success" : currentBuild.result
            def jobName = "${env.JOB_NAME}"

            // Strip the branch name out of the job name (ex: "Job Name/branch1" -> "Job Name")
            jobName = jobName.getAt(0..(jobName.indexOf('/') - 1))
        
            if (failed > 0) {
                buildStatus = "Failed"

                if (isPublishingBranch()) {
                    buildStatus = "MasterFailed"
                }

                buildColor = "danger"
                def failedTestsString = getFailedTests()

                notifySlack("", slackNotificationChannel, [
                    [
                        title: "${jobName}, build #${env.BUILD_NUMBER}",
                        title_link: "${env.BUILD_URL}",
                        color: "${buildColor}",
                        text: "${buildStatus}\n${author}",
                        "mrkdwn_in": ["fields"],
                        fields: [
                            [
                                title: "Branch",
                                value: "${env.BRANCH_NAME}",
                                short: true
                            ],
                            [
                                title: "Test Results",
                                value: "${testSummary}",
                                short: true
                            ],
                            [
                                title: "Last Commit",
                                value: "${message}",
                                short: false
                            ]
                        ]
                    ],
                    [
                        title: "Failed Tests",
                        color: "${buildColor}",
                        text: "${failedTestsString}",
                        "mrkdwn_in": ["text"],
                    ]
                ])          
            } else {
                notifySlack("", slackNotificationChannel, [
                    [
                        title: "${jobName}, build #${env.BUILD_NUMBER}",
                        title_link: "${env.BUILD_URL}",
                        color: "${buildColor}",
                        author_name: "${author}",
                        text: "${buildStatus}\n${author}",
                        fields: [
                            [
                                title: "Branch",
                                value: "${env.BRANCH_NAME}",
                                short: true
                            ],
                            [
                                title: "Test Results",
                                value: "${testSummary}",
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
        
        if (isPublishingBranch() && isResultGoodForPublishing()) {
            stage ('Publish') {
                sh "./gradlew ${gradleDefaultSwitches}"
            }
        }
    } catch (hudson.AbortException ae) {
        // I ignore aborted builds, but you're welcome to notify Slack here
    } catch (e) {
        def buildStatus = "Failed"

        if (isPublishingBranch()) {
            buildStatus = "MasterFailed"
        }

        notifySlack("", slackNotificationChannel, [
            [
                title: "${env.JOB_NAME}, build #${env.BUILD_NUMBER}",
                title_link: "${env.BUILD_URL}",
                color: "danger",
                author_name: "${author}",
                text: "${buildStatus}",
                fields: [
                    [
                        title: "Branch",
                        value: "${env.BRANCH_NAME}",
                        short: true
                    ],
                    [
                        title: "Test Results",
                        value: "${testSummary}",
                        short: true
                    ],
                    [
                        title: "Last Commit",
                        value: "${message}",
                        short: false
                    ],
                    [
                        title: "Error",
                        value: "${e}",
                        short: false
                    ]
                ]
            ]
        ])

        throw e
    }
}
