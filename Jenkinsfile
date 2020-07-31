pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build App') {
      steps {
        sh "mvn install"
      }
    }
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
             openshift.withProject('mapit-sample'){
            return !openshift.selector("bc", "mapit").exists();
          }
        }
        }
      }
      steps {
        script {
          openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') {
            openshift.newBuild("--name=mapit", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
          }
          }}
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') {
            openshift.selector("bc", "mapit").startBuild("--from-file=target/mapit-spring.jar", "--wait")
          }
          }}
      }
    }
    stage('Promote to DEV') {
      steps {
        script {
            openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') { 
              openshift.tag("mapit:latest", "mapit:dev")
          }
            }}
      }
    }
    stage('Create DEV') {
      when {
        expression {
            openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') {
          return !openshift.selector('dc', 'mapit-dev').exists()
          }
            }}
      }
      steps {
        script {
            openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') {
            openshift.newApp("mapit:latest", "--name=mapit-dev").narrow('svc').expose()
          }
            }}
      }
    }
    stage('Promote STAGE') {
      steps {
        script {
            openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') {
            openshift.tag("mapit:dev", "mapit:stage")
          }
        }
        }}
    }
    stage('Create STAGE') {
      when {
        expression {
            openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') {
              return !openshift.selector('dc', 'mapit-stage').exists()
          }
            }}
      }
      steps {
        script {
            openshift.withCluster('insecure://c101-e.jp-tok.containers.cloud.ibm.com:32240', 'qmnmLpoSjEDnFARAisTKH1QePfCsvdsyv4xrTVjH7YI') {
            openshift.withProject('mapit-sample') {
          openshift.newApp("mapit:stage", "--name=mapit-stage").narrow('svc').expose()
          }
            }}
      }
    }
  }
}
