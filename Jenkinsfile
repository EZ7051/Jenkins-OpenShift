#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        
        // Add steps here
        sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          openshift.withCluster() {
            openshift.withProject("ez7051-dev") {

              def buildConfigExists = openshift.selector("bc", "Jenkins-OpenShift").exists()

              if(!buildConfigExists){
                openshift.newBuild("--name=Jenkins-OpenShift", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
              }

              openshift.selector("bc", "Jenkins-OpenShift").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() {
            openshift.withProject("ez7051-dev") {
              def deployment = openshift.selector("dc", "Jenkins-OpenShift")

              if(!deployment.exists()){
                openshift.newApp('Jenkins-OpenShift', "--as-deployment-config").narrow('svc').expose()
              }

              timeout(5) {
                openshift.selector("dc", "Jenkins-OpenShift").related('pods').untilEach(1) {
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