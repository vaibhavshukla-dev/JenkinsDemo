#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Deploye Code') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"		    
                 def rc2 = bat (returnStdout: true, script: "git diff --name-only HEAD HEAD~1").trim()
		 //def rc2 = bat (returnStdout: true, script: "git diff --name-only HEAD HEAD~5").trim()
		 result = rc2.readLines().drop(1)
		    def folderString
		    for(int  i=0; i<result.size();i++){
			    folderString=''
		    	println("Res"+i+"->"+result[i])
			splittedParts = result[i].split('/')
			echo splittedParts[splittedParts.size()-1]
		    	for(int j=0; j<splittedParts.size()-1;j++){
			    folderString=folderString+splittedParts[j]+"\\"
		    	}
			echo folderString
		        rc3 = bat returnStatus: true, script: "mkdir C:\\deploy-cmp\\${folderString}"
		        correctstring = result[i].split('/').join('\\');
		    	rc4 = bat returnStatus: true, script: "copy ${correctstring} C:\\deploy-cmp\\${folderString}"
		    	rc5 = bat returnStatus: true, script: "copy ${correctstring}-meta.xml C:\\deploy-cmp\\${folderString}"			    
		    }
            }
            if (rc != 0) { error 'hub org authorization failed' }
			println rc
			// need to pull out assigned username
			if (isUnix()) {
				rmsg = sh returnStdout: true, script: "${toolbelt} force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}else{
			   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:deploy -u ${HUB_ORG} --sourcepath C:\\deploy-cmp\\force-app\\main\\default\\"
			}
            printf rmsg
            println('Deployment is Finished Successfully three!!')
            println(rmsg)
            rc5 = bat returnStatus: true, script: "cd c:\\deploy-cmp & rmdir /Q/S force-app"			    
        }
    }
}
