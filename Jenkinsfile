pipeline {
    
	agent any
/*	
	tools {
        maven "maven3"
    }
*/	
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.40.209:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	NEXUS_REPO_ID    = "vprofile-release"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	stage('SCA'){
            steps {
                dependencyCheck additionalArguments: '', odcInstallation: '6.5.0'
	    }
		post {
                success {
                    echo 'Now Archiving...'
                echo 'Risk Gate Thresholds...'
                dependencyCheckPublisher failedNewCritical: 1, failedNewHigh: 1, failedNewLow: 1, failedNewMedium: 1, failedTotalCritical: 1, failedTotalHigh: 1, failedTotalLow: 1, failedTotalMedium: 1, pattern: '**/dependency-check-report.xml', unstableNewCritical: 1, unstableNewHigh: 1, unstableNewLow: 1, unstableNewMedium: 1, unstableTotalCritical: 1, unstableTotalHigh: 1, unstableTotalLow: 1, unstableTotalMedium: 1
        		}
		}
	}	
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonarscanner4'
          }

          steps {
            withSonarQubeEnv('sonar-pro') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.organization=devsecops-sast \
	       	   -Dsonar.projectKey=sast-k \
                   -Dsonar.projectName=sast \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ '''
            }

            timeout(time: 20, unit: 'SECONDS') {
               waitForQualityGate abortPipeline: true
            }
          }
        }
    }


}
