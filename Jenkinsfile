pipeline {
  agent any
  tools {
  maven 'mvnF'
  }
    stages {

  stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/sandy2341/ksr-Full-Jenkins-pro.git']]])
      }
   }
	  
	stage ('Build')  {
	    steps {
        dir('app'){
            sh "mvn package"
          }
        }    
   }
   
  stage ('SonarQube Analysis') {
    steps {
      withSonarQubeEnv('sonar') {           
				dir('app'){
          sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=cicd_project'
        }
    }
    }
 }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "JFrog",
                    url: "https://sandya.jfrog.io/artifactory",
                    credentialsId: "JFrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "mvn01-libs-release-local",
                    snapshotRepo: "mvn01-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "default-maven-virtual",
                    snapshotRepo: "default-maven-virtual"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "mvnF", // Tool name from Jenkins configuration
                    pom: 'app/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

stage('Docker Build') {
      steps {
        script {
              docker.withRegistry( 'https://registry.hub.docker.com', 'dockerHub' ) {
              def customImage = docker.build("mydoc04/webapp")
              customImage.push()
          }
      }
    }
}
   stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

  stage('Build Helm Charts') {
    steps {
        dir('charts') {
        withCredentials([usernamePassword(credentialsId: 'JFrog', usernameVariable: 'username', passwordVariable: 'password')]) {
             sh 'sudo /usr/local/bin/helm package webapp'
             sh 'sudo /usr/local/bin/helm push-artifactory webapp-1.0.tgz https://sandya.jfrog.io/artifactory/ksr-helm-local --username $username --password $password'
		  }
        }
        }
      }

  } 
}