#!groovy
node {
    withEnv(['ONE=true',
             'SIX=sqlite',
             "DOCKER_IMAGE_NAME=my_image_name:${env.BUILD_ID}"
            ]) {

         stage("Checkout") {
             checkout scm
         }
         stage('Build') {
             echo "ONE is ${ONE}"
             echo "SIX is ${SIX}"
             sh 'printenv'

             docker.withRegistry("${DOCKER_REGISTRY_HOST}",'docker_registry_auth') {
                 def image = docker.build("${DOCKER_IMAGE_NAME}")
                 image.push()
             }
         }
    }
}
