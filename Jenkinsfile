#! /usr/bin/env groovy

pipeline {
  agent {
    label 'maven'
  }
  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'

        script {
          openshift.withCluster() {
            openshift.withProject("poc-santander") {
                def buildConfigExists = openshift.selector("bc", "javaapp").exists()

                if(!buildConfigExists){
                    openshift.newBuild("--name=javaapp", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
                }
                openshift.selector("bc", "javaapp").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow")
            }
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {
          openshift.withCluster() {
            openshift.withProject("poc-santander") {
              def deployment = openshift.selector("dc", "javaapp")
              if(!deployment.exists()){
                openshift.newApp('javaapp', "--as-deployment-config").narrow('svc').expose()
              }

              timeout(5) {
                openshift.selector("dc", "javaapp").related('pods').untilEach(1) {
                  return (it.object().status.phase == "Running")
                  }
                }
            }
          }
        }
      }
    }
  }
}