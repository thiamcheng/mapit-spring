pipeline {
  agent {
      label 'maven'
    }
  stages {
    
      stage ('Artifactory configuration') {
          steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "http://161.202.190.34:8082/artifactory/",
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
    
         
        stage('SonarQube Code Analysis') {

          steps {
          script{

           //  withSonarQubeEnv('sonarscanner') {
           // def mvnHome = tool 'maven'
           // sh "'${mvnHome}/bin/mvn' org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -Dsonar.host.url=$SONAR_HOST_URL  -Dsonar.projectKey=${JOB_NAME} -Dsonar.projectName=${JOB_NAME} -Dsonar.language=java -Dsonar.sources=. -Dsonar.java.binaries=target -Dsonar.tests=. -Dsonar.test.inclusions=**/*Test*/* -Dsonar.exclusions=target/**/*.class"
           // }
               withMaven(maven : 'maven') {
                sh "mvn sonar:sonar"
            }
          
          }

          }
        }

     stage ('Build App') {
            steps {
                rtMavenRun (
                    tool: 'maven', // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }
    
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
             openshift.withProject('mapit-sample'){
            return !openshift.selector("bc", "mapit").exists();
          }
        }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject('mapit-sample') {
            openshift.newBuild("--name=mapit", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
          }
          }}
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject('mapit-sample') {
            openshift.selector("bc", "mapit").startBuild("--from-file=target/mapit-spring.jar", "--wait")
          }
          }}
      }
    }
    stage('Promote to DEV') {
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject('mapit-sample') { 
              openshift.tag("mapit:latest", "mapit:dev")
          }
            }}
      }
    }
    stage('Create DEV') {
      when {
        expression {
            openshift.withCluster() {
            openshift.withProject('mapit-sample') {
          return !openshift.selector('dc', 'mapit-dev').exists()
          }
            }}
      }
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject('mapit-sample') {
            openshift.newApp("mapit:latest", "--name=mapit-dev").narrow('svc').expose()
          }
            }}
      }
    }
    stage('Promote STAGE') {
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject('mapit-sample') {
            openshift.tag("mapit:dev", "mapit:stage")
          }
        }
        }}
    }
    stage('Create STAGE') {
      when {
        expression {
            openshift.withCluster() {
            openshift.withProject('mapit-sample') {
              return !openshift.selector('dc', 'mapit-stage').exists()
          }
            }}
      }
      steps {
        script {
            openshift.withCluster() {
            openshift.withProject('mapit-sample') {
          openshift.newApp("mapit:stage", "--name=mapit-stage").narrow('svc').expose()
          }
            }}
      }
    }
  }
}
