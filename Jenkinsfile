// Obtaining an Artifactory server instance defined in Jenkins:
			
def server = Artifactory.server 'artifactory1'

		 //If artifactory is not defined in Jenkins, then create on:
		// def server = Artifactory.newServer url: 'Artifactory url', username: 'username', password: 'password'

//Create Artifactory Maven Build instance
def rtMaven = Artifactory.newMavenBuild()

def buildInfo

pipeline {
    agent any

	tools {
		jdk "OpenJDK-11.0.2-x64"
		maven "Maven instalacija v3.6.1"
	}

    stages {
        stage('Clone sources'){
            steps {
                git url: 'https://github.com/balafr3/web_ex'
            }
        }

 /*    	stage('SonarQube analysis') {
	     steps {
		//Prepare SonarQube scanner enviornment
		withSonarQubeEnv('LocalSonar') {
		   sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.6.0.1398:sonar'
		}
	      }
	} */

//	stage('Quality Gate') {
//		steps {
//			timeout(time: 1, unit: 'HOURS') {
			//Parameter indicates wether to set pipeline to UNSTABLE if Quality Gate fails
		        // true = set pipeline to UNSTABLE, false = don't
			// Requires SonarQube Scanner for Jenkins 2.7+
//			waitForQualityGate abortPipeline: false
//		       }
//		 }
//	}

	stage('Artifactory configuration') {
		
	   steps {
		script {
			rtMaven.tool = 'Maven 3.6.1' //Maven tool name specified in Jenkins configuration
		
			rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server //Defining where the build artifacts should be deployed to
			
			rtMaven.resolver releaseRepo:'libs-release', snapshotRepo: 'libs-snapshot', server: server //Defining where Maven Build should download its dependencies from
			
			rtMaven.deployer.artifactDeploymentPatterns.addExclude("pom.xml") //Exclude artifacts from being deployed
			
			//rtMaven.deployer.deployArtifacts =false // Disable artifacts deployment during Maven run
		    
			buildInfo = Artifactory.newBuildInfo() //Publishing build-Info to artifactory
	
			/* buildInfo.retention maxBuilds: 10, maxDays: 7, deleteBuildArtifacts: true */
			
			buildInfo.env.capture = true
			}
	    }
	}

	stage('Execute Maven') {
		steps {
		   script {
		
		rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
			}
		}
		
	}

	stage('Publish build info') {
		steps {
		  script {

		server.publishBuildInfo buildInfo
		}
		}
	}

}
	post {
success {
mattermostSend color: 'good', message: 'Web_Ex succeded', endpoint: 'http://10.10.12.48:8065/hooks/8xewsmxtziro3cc7kyih4c3tcw', channel: 'build-info'
}
unstable {
mattermostSend color: 'warning', message: 'Web_Ex is unstable', endpoint: 'http://10.10.12.48:8065/hooks/8xewsmxtziro3cc7kyih4c3tcw', channel: 'build-info'
}
failure {
mattermostSend color: 'danger', message: 'Web_Ex has failed', channel: 'build-info', endpoint: 'http://10.10.12.48:8065/hooks/8xewsmxtziro3cc7kyih4c3tcw', icon: 'https://talks.bitexpert.de/zendcon16-jenkins-for-php-projects/images/jenkins.png'
}
changed {
mattermostSend color: 'warning', message: 'Web_Ex has changed', endpoint: 'http://10.10.12.48:8065/hooks/8xewsmxtziro3cc7kyih4c3tcw', channel: 'build-info'
}
}
}
