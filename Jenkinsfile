#!groovy
 
import groovy.json.JsonSlurperClassic

node {


if(env.BRANCH_NAME == 'main')
   scratchOrg env.BRANCH_NAME

}
 
def scratchOrg(branchName) {
 
    def SF_CONSUMER_KEY= "3MVG9KI2HHAq33Ry2vLlK9hC672.qmEadb86CU6xjlDirLW1zs.tzAbSxqk84_r1SivpGBuauCXPJBGms20Ed"
    def SF_USERNAME= "amuppidi@lightning-ccc.com"
    def SERVER_KEY_CREDENTALS_ID= env.SERVER_KEY_CREDENTALS_ID
    def TEST_LEVEL='RunLocalTests'
    def PACKAGE_VERSION
    def PACKAGE_NAME = "ebike_dev"
    def SF_INSTANCE_URL = "https://login.salesforce.com"
    def  sfdx='\"C:\\Program Files\\Salesforce CLI\\bin\\sfdx.cmd\"'
  
  
   
 
    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------
 
    stage('checkout source') {
        checkout scm
        
    }
  
  
    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
 
    withEnv(["HOME=${env.WORKSPACE}"]) {
     withCredentials([file(credentialsId:  'JWT'	, variable: 'server_key_file')]) {     
          stage('SFDX Update'){
          timeout(time: 3, unit: 'MINUTES') {
        
           rc = command "${sfdx} update" 
           if(rc!=0){
           
            error 'SFDX update failed'
           }
          }
          }
          stage('Autorize DevHub'){
           rc = command "${sfdx} auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file}  --setalias testHubOrg"
           if (rc != 0) {
            error 'Salesforce dev hub org authorization failed.'
           }
           
          }
      
      
            // -------------------------------------------------------------------------
            // Create new scratch org to test your code.
            // -------------------------------------------------------------------------
 
            stage('Create Test Scratch Org') {
                rc = command "${sfdx} force:org:create --targetdevhubusername testHubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ebike_scratch --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce test scratch org creation failed.'
                }
            }

            
            // -------------------------------------------------------------------------
            // Display test scratch org info.
            // -------------------------------------------------------------------------
 
            stage('Display Test Scratch Org') {
                rc = command "${sfdx} force:org:display --targetusername ebike_scratch"
                command"${sfdx} force:org:open --targetusername ebike_scratch"
                if (rc != 0) {
                    error 'Salesforce test scratch org display failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Push source to test scratch org.
            // -------------------------------------------------------------------------
 
            stage('Push To Test Scratch Org') {
                rc = command "${sfdx} force:source:push --targetusername ebike_scratch"
                if (rc != 0) {
                    error 'Salesforce push to test scratch org failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Run unit tests in test scratch org.
            // -------------------------------------------------------------------------
 
            stage('Run Tests In Test Scratch Org') {
                rc = command "${sfdx} force:apex:test:run --targetusername ciorg --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
                if (rc != 0) {
                    error 'Salesforce unit test run in test scratch org failed.'
                }
            }

            
            // -------------------------------------------------------------------------
            // Run unit tests in test scratch org.
            // -------------------------------------------------------------------------
 
            stage('Run Tests In Test Scratch Org') {
                rc = command "${sfdx} force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername HubOrg"
                
                // Wait 5 minutes for package replication
                 sleep 300

                 def jsonSlurper = new JsonSlurperClassic()
                 def response = jsonSlurper.parseText(output)
                 PACKAGE_VERSION = response.result.SubscriberPackageVersionId
                 response = null
                 echo ${PACKAGE_VERSION}

                if (rc != 0) {
                    error 'Salesforce Package creation failed.'
                }
            }
 
 
 

      
     }
    }
}
 
 def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }
 }  
 