pipeline {
    environment {
        ACR_REPO    = 'mstrdevopsworkshop'
        GIT_REPO = "https://github.com/ozcankahraman/cpx-oss-workshop-1.git"
        WEB_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-web"
        API_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-api"
        DB_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-db"
    }
  options { 
      disableConcurrentBuilds() 
      timestamps()
  }
  agent any
  stages {
      stage('Checkout') {
        steps {
            git url: "${env.GIT_REPO}", branch: 'master'
        }
      }
    stage('Build images') {
            steps{
                dir('app/api')
                {
                    script{
                    docker.build("${env.API_IMAGE}:${env.BUILD_NUMBER}")
                    }
                }
                dir('app/web')
                {
                   sh "sudo docker build --build-arg BUILD_DATE=`date '+%Y-%m-%dT%H:%M:%SZ'` --build-arg VCS_REF=`git rev-parse --short HEAD` --build-arg IMAGE_TAG_REF=${env.BUILD_NUMBER} -t ${env.WEB_IMAGE}:${env.BUILD_NUMBER} ."
                }
                dir('app/db')
                {
                    script{
                    docker.build("${env.DB_IMAGE}:${env.BUILD_NUMBER}")
                    }
                }
            }
       }
      stage('Test api image') {
         agent { docker "${env.API_IMAGE}:${env.BUILD_NUMBER}" } 
               steps {
                        echo '$HOSTNAME'
                     }
        }
      
       stage('Test web image') {
         agent { docker "${env.WEB_IMAGE}:${env.BUILD_NUMBER}" } 
               steps {
                        echo '$HOSTNAME'
                     }
      }
      
        stage('Test mongo image') {
         agent { docker "${env.DB_IMAGE}:${env.BUILD_NUMBER}" } 
               steps {
                        echo 'test'
                     }
      }
    stage('Push images to ACR') {
        steps{
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'acr-credentials',usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
            sh "docker login ${env.ACR_LOGINSERVER} -u $USERNAME -p $PASSWORD"
            sh "docker push ${env.API_IMAGE}:${env.BUILD_NUMBER}"
            //sh "docker tag ${env.API_IMAGE}:${env.BUILD_NUMBER} ${env.API_IMAGE}:latest"
            //sh "docker push ${env.API_IMAGE}:latest"
            sh "docker login ${env.ACR_LOGINSERVER} -u $USERNAME -p $PASSWORD"
            sh "docker push ${env.WEB_IMAGE}:${env.BUILD_NUMBER}"
            //sh "docker tag ${env.WEB_IMAGE}:${env.BUILD_NUMBER} ${env.WEB_IMAGE}:latest"
            //sh "docker push ${env.WEB_IMAGE}:latest"
            sh "docker login ${env.ACR_LOGINSERVER} -u $USERNAME -p $PASSWORD"
            sh "docker push ${env.DB_IMAGE}:${env.BUILD_NUMBER}"
            //sh "docker tag ${env.DB_IMAGE}:${env.BUILD_NUMBER} ${env.DB_IMAGE}:latest"
            //sh "docker push ${env.DB_IMAGE}:latest"
            }
        }
    }
    stage('Helm PreSteps'){
          steps{
            sh 'sudo /usr/local/bin/helm init --client-only' 
          }
    }
    stage('Deploy using Helm') {
        steps{
               dir('app')
               {
                   sh "sudo /usr/local/bin/helm upgrade --install dbhelmdb ./dbchart/ --kubeconfig /home/azureuser/.kube/config --set dbserver.image.repo=${env.DB_IMAGE} --set dbserver.image.tag=${env.BUILD_NUMBER}"
                   sh "sudo /usr/local/bin/helm upgrade --install webapihelmd ./webapichart/ --wait --kubeconfig /home/azureuser/.kube/config --set webserver.image.repo=${env.WEB_IMAGE} --set webserver.image.tag=${env.BUILD_NUMBER} --set apiserver.image.repo=${env.API_IMAGE} --set apiserver.image.tag=${env.BUILD_NUMBER}"
               }
        }
    }
  }
}
