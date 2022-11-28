pipeline{

  agent {
      kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: joseandresdevops/nodonodejs:1.0
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
        '''

        defaultContainer 'shell'
      }

  }

  environment {
    registryCredential='docker-hub-credentials'
    registryFrontend = 'joseandresdevops/angular-14-app'
  }


  stages {
    stage('Build') {
      steps {

        sh 'npm install'
        sh 'npm run build &'
        sh 'npm package'
        sleep 20
      }
    }



    stage('Push Image to Docker Hub') {
      steps {
        script {

          dockerImage = docker.build registryFrontend + ":$BUILD_NUMBER"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }



    stage('Push Image latest to Docker Hub') {
      steps {
        script {

          dockerImage = docker.build registryFrontend + ":latest"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()

          }
        }
      }
    }



    stage('Deploy to K8s') {
      steps{
        script {

          if(fileExists("configuracion")){
            sh 'rm -r configuracion'
          }
        }

        sh 'git clone https://github.com/JoseAndresDevOps/kubernetes-helm-docker-config.git configuracion --branch test-implementation'
        sh 'kubectl apply -f configuracion/kubernetes-deployment/angular-14-app/manifest.yml -n default --kubeconfig=configuracion/kubernetes-config/config'
      }
    }


  }

  post {
    always {
      sh 'docker logout'
    }
  }

}