pipeline {
  agent {
      label 'maven'
    }
	
   environment {
	   DEPLOY_NS = "${env.DEPLOY_NS}"
        
    }
  stages {
    
      stage ('Echoing valuesp') {
            steps {
		    
		    openshift.withCluster() {
		          echo "DEPLOY_NS" + "${env.DEPLOY_NS}"
			  echo "Selector Project result" + openshift.selector('project', "${DEPLOY_NS}").exists()
			  echo "Selector ns result" + openshift.selector('ns', "${DEPLOY_NS}").exists()
			  echo "Selector all result" + openshift.selector('all', "${DEPLOY_NS}").exists()

		    }
	        }
            }
        }
	  
    stage('Deleting project if any ..') {
      when {
             expression {
                 openshift.withCluster() {
			 return openshift.selector('project', "${DEPLOY_NS}").exists()
		  }	
	     }
        } 
      steps {
        script {
            openshift.withCluster() {
	       echo "Deleting the project"   
	      openshift.raw("delete project ${DEPLOY_NS}") 
              }
	}
      }	 
    }
	    
        
	
	  
      stage ('Artifactory configuration') {
          steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "${env.ARTIFACTORY_SERVER_ID}",
                    credentialsId: "jfrog-credentials"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
            }
        } 
    
       stage ('Maven Clean App') {
            steps {
                rtMavenRun (
                    tool: 'maven', // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }
      
         
        stage('SonarQube Code Analysis') {

          steps {
          script{

           //  withSonarQubeEnv('sonarscanner') {
           // def mvnHome = tool 'maven'
           // sh "'${mvnHome}/bin/mvn' org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -Dsonar.host.url=$SONAR_HOST_URL  -Dsonar.projectKey=${JOB_NAME} -Dsonar.projectName=${JOB_NAME} -Dsonar.language=java -Dsonar.sources=. -Dsonar.java.binaries=target -Dsonar.tests=. -Dsonar.test.inclusions=**/*Test*/* -Dsonar.exclusions=target/**/*.class"
           // }
               withMaven(maven : 'maven') {
                sh "mvn sonar:sonar -Dsonar.language=java -Dsonar.java.binaries=target"
            }
          
          }

          }
        }

     stage ('Maven Install App') {
            steps {
                rtMavenRun (
                    tool: 'maven', // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }
		
  stage('Creating project if any ..') {
      steps {
            script {
                    openshift.withCluster() {	 
	
			     try {
				   openshift.withCredentials( 'Jenkins01-token' ) {
            			   openshift.newProject( "${DEPLOY_NS}" )
                                 // ...
                             }
                            } catch ( e ) {
        			// The exception is a hudson.AbortException with details
        				// about the failure.
        			"Error encountered: ${e}"
 			   }

			    
                }
	}
      }
    }	
       
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
             openshift.withProject("${DEPLOY_NS}"){
            return !openshift.selector("bc", "mapit").exists();
          }
        }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") {
            openshift.newBuild("--name=mapit", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
          }
          }}
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") {
            openshift.selector("bc", "mapit").startBuild("--from-file=target/mapit-spring.jar", "--wait")
          }
          }}
      }
    }
    stage('Promote to DEV') {
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") { 
              openshift.tag("mapit:latest", "mapit:dev")
          }
            }}
      }
    }
    stage('Create DEV') {
      when {
        expression {
            openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") {
          return !openshift.selector('dc', 'mapit-dev').exists()
          }
            }}
      }
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") {
            openshift.newApp("mapit:latest", "--name=mapit-dev").narrow('svc').expose()
          }
            }}
      }
    }
    stage('Promote STAGE') {
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") {
            openshift.tag("mapit:dev", "mapit:stage")
          }
        }
        }}
    }
    stage('Create STAGE') {
      when {
        expression {
            openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") {
              return !openshift.selector('dc', 'mapit-stage').exists()
          }
            }}
      }
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS}") {
          openshift.newApp("mapit:stage", "--name=mapit-stage").narrow('svc').expose()
          }
            }}
      }
    }
  }
}

