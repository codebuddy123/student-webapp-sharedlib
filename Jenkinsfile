
    node {

    def mavenHome = tool name: 'maven-3.9.10', type: 'maven'
   
    // Pulling the code from SCM repository
    stage("Git checkout") {
        checkout scmGit(
            branches: [[name: '*/main']],
            userRemoteConfigs: [[
                credentialsId: 'ce4402e7-050c-46c0-b476-9b9e33e8db44',
                url: 'https://github.com/codebuddy123/student-reg-webapp.git'
            ]]
        )
    }
    
    // Building the war package using Maven build tool
    stage("Building the package") {
        sh "${mavenHome}/bin/mvn clean package"
    }

    // Scanning the Code using SAST tool Sonarqube
    stage("Sonarqube Analysis") {
        sh "${mavenHome}/bin/mvn sonar:sonar" 
    }

    // Pushing the artifact to Nexus repo
    stage("Storing the artifact to Nexus") {
        sh "${mavenHome}/bin/mvn deploy"
    }
    
    // Stopping the Tomcat Service Before the WAR deployment
    stage("stopping tomcat") {
        sshagent(['tomcat-server']) {
            sh """
                echo Stopping the Tomcat Service
                ssh -o StrictHostKeyChecking=no ec2-user@172.31.38.104 'sudo systemctl stop tomcat'
                sleep 5
            """
        }
    }

    // Deploying the artifact into Tomcat Server
    stage("Tomcat Deployment") {
        sshagent(['tomcat-server']) {
            //removing old war file
            sh "rm -rf /opt/tomcat/webapps/student-reg-webapp.war"
            sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@172.31.38.104:/opt/tomcat/webapps"
        }
    }

    // Starting the Tomcat Service after the WAR Deployment
    stage("starting tomcat") {
        sshagent(['tomcat-server']) {
            sh """
                echo Starting the Tomcat Service
                ssh -o StrictHostKeyChecking=no ec2-user@172.31.38.104 'sudo systemctl start tomcat'
                echo Successfully started tomcat
            """
        }
    }

}