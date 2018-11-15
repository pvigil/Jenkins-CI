#!groovy
import groovy.json.JsonSlurperClassic
node {
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def QA_HUB_ORG= env.QA_HUB_ORG_DH
    def QA_SFDC_HOST = env.SANDBOX_SFDC_HOST
    def QA_JWT_KEY_CRED_ID = env.QA_JWT_CRED_ID_DH
    def QA_CONNECTED_APP_CONSUMER_KEY= env.QA_CONNECTED_APP_CONSUMER_KEY_DH
    println 'KEY IS' 
    println QA_JWT_KEY_CRED_ID
    def toolbelt = tool 'toolbelt'

    //stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        //checkout scm
   //}

    withCredentials([file(credentialsId: QA_JWT_KEY_CRED_ID, variable: 'jwt_key_file'), string(credentialsId: QA_CONNECTED_APP_CONSUMER_KEY, variable: 'consumer_key'), string(credentialsId: QA_HUB_ORG, variable: 'hub_org')]) {
        stage('Connect to QA') { 
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${consumer_key} --username ${hub_org} --jwtkeyfile ${jwt_key_file} --setalias qa --instanceurl ${QA_SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${consumer_key} --username ${hub_org} --jwtkeyfile \"${jwt_key_file}\" --setdefaultusername --setalias qa --instanceurl ${QA_SFDC_HOST}"
            }
            if (rc != 0) { error 'Connection failed' } 
            
        }
        
          stage('Deploy to QA') {
              if (isUnix()) {
                    rc = sh returnStatus: true, script: "${toolbelt} force:mdapi:deploy -d ./ -u qa -l RunLocalTests -w -1"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" force:mdapi:deploy -d ./ -u qa -l RunLocalTests -w -1"
              }
            if (rc != 0) {
                error 'Deploy failed'
            }
            
          }
             
    }
} 
