node {

    stage ("checkout")  {
      git branch: 'main', credentialsId: 'Github', url: 'https://github.com/danebuz08/my-javawebapp-repo.git'
    }
    
    stage("build") {
        def mvnHome = tool 'maven'
        sh "${mvnHome}/bin/mvn clean install -f MyWebApp/pom.xml"
    }
    stage("unit tests and coverage") {
        junit '**/target/surefire-reports/*.xml'
        jacoco()
    }
    stage ("code analysis") {
      def mvnHome = tool 'maven'
      
      withSonarQubeEnv("sonarqube") {
         sh "${mvnHome}/bin/mvn sonar:sonar -f MyWebApp/pom.xml"
      }
     stage ("SAST scan"){
         sh "trivy fs ./MyWebApp"
     }
     stage ("binary upload") {
         nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: 'Nexus', groupId: 'com.dept.app', nexusUrl: '3.147.212.124:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
     }
     stage ("Deploy to Dev") {
         deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tomcat', path: '', url: 'http://16.59.9.106:8081')], contextPath: null, war: '**/*.war'
     }
     stage ("DEV notify") {
         slackSend channel: 'qa-testing-team', message: 'Hey Dev team - Deployment is done, please start smoke testing.'
     }
     
     //CI pipeline ends here
     /**
      * CD pipeline starts here
      */
      stage ("DEV approve") {
          echo "taking approval from DEV Manager for QA Deployment"
          timeout(time:7, unit: 'DAYS') {
                input message: 'Do you approve QA Deployment?', submitter: 'admin'
          }
      }
      
      stage ("QA deploy") {
          deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tomcat', path: '', url: 'http://16.59.9.106:8081')], contextPath: null, war: '**/*.war'
      }
      stage ("QA notify") {
         slackSend channel: 'qa-testing-team', message: 'Hey QA team - QA Deployment is done, please start functional testing.'
      }
      stage ("QA approve") {
          echo "taking approval from QA Manager for PROD Deployment"
          timeout(time:2, unit: 'DAYS') {
                input message: 'Do you approve PROD Deployment?', submitter: 'admin'
          }
      }
      stage ("PROD deploy") {
          deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tomcat', path: '', url: 'http://16.59.9.106:8081')], contextPath: null, war: '**/*.war'
      }
      stage ("final notify") {
         slackSend channel: 'qa-testing-team', message: 'Hey PO team - PROD Deployment is done please inform end customers.'
      }
    }
}

