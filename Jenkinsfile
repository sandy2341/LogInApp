pipeline {
  agent any
  tools {
  maven 'mvnf'
  }
    stages {

  stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/sandy2341/LogInApp.git']]])
      }
   }
	  
	stage ('Build')  {
	    steps {
        dir('LogInApp'){
            sh "/opt/mvnf/bin/mvn package"
          }
        }    
   }
   
  stage ('SonarQube Analysis') {
    steps {
      withSonarQubeEnv('sonar') {           
				dir('LogInApp'){
          sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=ksrproject123'
        }
    }
    }
 } 

  } 
}
