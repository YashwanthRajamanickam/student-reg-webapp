node {
    try{
    def Tomcat_Ip='35.154.72.248'
    def mvnhome=tool name: 'Maven-3.9.11', type: 'maven'
    stage('Clone') {
               git branch: 'development', credentialsId: 'Yashwanth_Studentwebapp', url: 'https://github.com/YashwanthRajamanickam/student-reg-webapp.git'
    }
    
    stage('Build'){
        sh """
            ${mvnhome}/bin/mvn clen package 
            echo "build success"
            """
    }
     stage('test'){
        withCredentials([string(credentialsId: 'SonarQube', variable: 'SonarQube')]) {
    sh """
            ${mvnhome}/bin/mvn sonar:sonar -Dsonar.token=${SonarQube}
            echo "Test success"
            """
}
     }
     stage('deploy to nexus'){
            sh """
            ${mvnhome}/bin/mvn deploy 
            echo "deploy success"
            """
    }
    stage('Stop Tomcat'){
         sshagent(['Tomcat']) {
     sh """
            ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_Ip} '
                cd /opt/tomcat/bin && sudo ./shutdown.sh
            '
        """
    }
    }
   
    stage('Deploy to Tomcat'){
        sshagent(['Tomcat']) {
         sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${Tomcat_Ip}:/opt/tomcat/webapps/"

}
    }
    stage('Start Tomcat'){
         sshagent(['Tomcat']) {
     sh """
            ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_Ip} '
                cd /opt/tomcat/bin && sudo ./startup.sh
            '
        """
    }
}
    }
     catch(err){
        echo "The build has failed due to an error: ${err.getMessage()}"
        currentBuild.result = 'FAILURE'
        def buildStatus = currentBuild.currentResult
        emailext body: "The Build for ${env.JOB_NAME} has been Failed and please check the logs on ${env.BUILD_URL}", subject: "${env.BUILD_NUMBER} - ${env.JOB_NAME} - Build is ${buildStatus}", to: 'yashwanthr2498@gmail.com'
        slackSend channel: 'devops-operations', color: 'danger', message: "The Build for ${env.JOB_NAME} has been Failed and please check the logs on ${env.BUILD_URL} .More info-${env.BUILD_NUMBER} - ${env.JOB_NAME} - Build is ${buildStatus}"
    }
//     finally{
        
//     def buildStatus = currentBuild.currentResult
//     if (buildStatus == 'SUCCESS') {
//     emailext body: "The Build for ${env.JOB_NAME} has been Passed and please check the logs on ${env.BUILD_URL}", subject: "${env.BUILD_NUMBER} - ${env.JOB_NAME} - Build is ${buildStatus}", to: 'yashwanthr2498@gmail.com'
//     }else {
//     emailext body: "The Build for ${env.JOB_NAME} has been Failed and please check the logs on ${env.BUILD_URL}", subject: "${env.BUILD_NUMBER} - ${env.JOB_NAME} - Build is ${buildStatus}", to: 'yashwanthr2498@gmail.com'
//     }

//     if (buildStatus == 'SUCCESS') {
//      slackSend channel: 'devops-operations', color: 'good', message: "The Build for ${env.JOB_NAME} has been Passed and please check the logs on ${env.BUILD_URL} .More info-${env.BUILD_NUMBER} - ${env.JOB_NAME} - Build is ${buildStatus}"
//       }
//     else {
//      slackSend channel: 'devops-operations', color: 'danger', message: "The Build for ${env.JOB_NAME} has been Failed and please check the logs on ${env.BUILD_URL} .More info-${env.BUILD_NUMBER} - ${env.JOB_NAME} - Build is ${buildStatus}"
 
//       }
    

    
// }

finally{
    def buildStatus = currentBuild.currentResult
    if (buildStatus == 'SUCCESS'){
    emailext body: 
    """"<b>Build</b> Successful!"
Job : "${env.JOB_NAME}
Build Number : #${env.BUILD_NUMBER}
Logs : ${env.BUILD_URL}""", subject: "Jenkins Build ${buildStatus} : ${env.JOB_NAME} #${env.BUILD_NUMBER}", to: 'yashwanthr2498@gmail.com'
    }else{
     if (buildStatus != 'SUCCESS'){
    emailext (body: 
    """<p><b>Build Failure</b></p>
<p>Job : "${env.JOB_NAME}</p>
<p>Build Number : #${env.BUILD_NUMBER}</p>
<p>Logs : <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""", subject: "Jenkins Build ${buildStatus} : ${env.JOB_NAME} #${env.BUILD_NUMBER}", to: 'yashwanthr2498@gmail.com'
mimeType: 'text/html'
    )
    }
    }
}
}