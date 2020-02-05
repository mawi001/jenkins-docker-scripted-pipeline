#!groovy
pipeline {
    agent any
    environment {
        dockerImageName = "user/my_image:${env.BUILD_NUMBER}"
        slack_channel   = "#ci-channel"
    }

    def tryStep(String message, Closure block, Closure tearDown = null) {
        try {
            block()
        }
        catch (Throwable t) {
            slackSend message: "${env.JOB_NAME}: ${message} failure ${env.BUILD_URL}", channel: "${env.slack_channel}", color: 'danger'

            throw t
        }
        finally {
            if (tearDown) {
                tearDown()
            }
        }
    }

    stages {
      node {
          stage("Checkout") {
              checkout scm
          }

          stage('Test') {
              tryStep "test", {
                  sh "ls Dockerfile"

              },
              {
                  sh "cat Dockerfile"
              }
          }

          stage("Build docker image from Dockerfile") {
              tryStep "build", {
                  docker.withRegistry("${docker_registry_host}",'docker_registry_auth') {
                      def image = docker.build("${env.dockerImageName}",'.')
                      image.push()
                  }
              }
          }
      }

      String BRANCH = "${env.BRANCH_NAME}"

      if (BRANCH == "develop") {

          node {
              stage('Push develop image') {
                  tryStep "image tagging", {
                      docker.withRegistry("${docker_registry_host}",'docker_registry_auth') {
                          def image = docker.build("${env.dockerImageName}",'.')
                          image.pull()
                          image.push("develop")
                      }
                  }
              }
          }
      }

      if (BRANCH == "master") {

          node {
              stage('Push acceptance image') {
                  tryStep "image tagging", {
                      docker.withRegistry("${docker_registry_host}",'docker_registry_auth') {
                          def image = docker.build("${env.dockerImageName}",'.')
                          image.pull()
                          image.push("acceptance")
                      }
                  }
              }
          }

          node {
              stage("Deploy to ACC") {
                  tryStep "deployment", {
                      build job: 'Subtask_Openstack_Playbook',
                          parameters: [
                              [$class: 'StringParameterValue', name: 'INVENTORY', value: 'acceptance'],
                              [$class: 'StringParameterValue', name: 'PLAYBOOK', value: 'deploy-gob-workflow.yml'],
                          ]
                  } # end step
              } # end stage
          } #end node
      } # end branch master
    } # end stages
} # end pipeline
