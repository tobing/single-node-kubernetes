pipeline {

  environment {
    dockerimagename = "php:8-fpm-alpine"
    dockerImage = ""
  }

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/tobing/single-node-kubernetes.git'
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'dockerhublogin'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "local_persistentvolume.yml", "nginx_configMap.yaml", "nginx_deployment.yaml", "nginx_hpa.yaml", "nginx_service.yaml", "php_deployment.yaml", "php_hpa.yaml", "php_service.yaml", kubeconfigId: "kuberneteslogin")
        }
      }
    }

  }

}
