pipeline {
  agent any
  stages {
    stage('Install packages') {
      steps {
        sh 'npm install'
      }
    }

    stage('Lint') {
      steps {
        sh 'npm run lint'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
          dockerImageLatest = docker.build registry + ":latest"
        }

      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
            dockerImageLatest.push()
          }
        }

      }
    }

    stage('Remove Unused docker image') {
      steps {
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }

    stage('Building AWS infrastructure') {
      steps {
        withCredentials(bindings: [[ $class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Aws-Capstone', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
          sh 'ansible-playbook ./ansible/ec2-launcher.yml'
        }

      }
    }

    stage('Configure kubernetes cluster') {
      steps {
        sh 'ansible-playbook -i ./ansible/inventory -v ./ansible/ec2-configure.yml'
      }
    }

    stage('Deploying app containers on kubernetes') {
      steps {
        sh 'ansible-playbook -i ./ansible/inventory -v ./ansible/k8s-deploy.yml'
      }
    }

  }
  environment {
    registry = 'metanitesh/simple-api'
    registryCredential = 'dockerId'
    dockerImage = ''
    dockerImageLatest = ''
  }
}