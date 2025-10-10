node {
    try{
    def Tomcat_Ip='13.234.67.39'
    def mvnhome=tool name: 'Maven-3.9.11', type: 'maven'
    stage('Clone') {
               git branch: 'development', credentialsId: 'Yashwanth_Studentwebapp', url: 'https://github.com/YashwanthRajamanickam/student-reg-webapp.git'
    }
    
    stage('Build'){
        sh """
            ${mvnhome}/bin/mvn clean package 
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
    catch(Exception e){
        sh "echo 'The build is failed:${e.getMessage()}'"
        currentBuild.result="FAILURE"
    }
    finally{
    def buildStatus = currentBuild.currentResult
    if (buildStatus == 'SUCCESS') {
    emailext body: "${env.JOB_NAME}. 
    The Build has been Passed and please check the logs on ${env.BUILD_URL}", 
    subject: "${env.BUILD_NUMBER}", to: "yashwanthr2498@gmail.com"
    }else {
     emailext body: "${env.JOB_NAME}. 
    The Build has been Failed and please check the logs on ${env.BUILD_URL}", 
    subject: "${env.BUILD_NUMBER}", to: "yashwanthr2498@gmail.com"
    }
    
}