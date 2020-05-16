pipeline {

	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e website/*.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						 docker build -t jay13patel/capstone .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						 docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						 docker push jay13patel/capstone

					'''
  			}

      }
    }

    stage('Set current kubectl context') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'eks-cred') {
          sh '''
						kubectl config use-context arn:aws:eks:us-east-1:315002452909:cluster/finalcluster
					'''
        }

      }
    }

    stage('Deploy blue container') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'eks-cred') {
          sh '''
						kubectl apply -f ./blue-controller.json
					'''
        }

      }
    }

    stage('Deploy green container') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'eks-cred') {
          sh '''
						kubectl apply -f ./green-controller.json
					'''
        }

      }
    }

    stage('Create the service in the cluster, redirect to blue') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'eks-cred') {
          sh '''
						kubectl apply -f ./blue-service.json
					'''
        }

      }
    }

    stage('Wait user approve') {
      steps {
        input 'Ready to redirect traffic to green?'
      }
    }

    stage('Create the service in the cluster, redirect to green') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'eks-cred') {
          sh '''
						kubectl apply -f ./green-service.json
					'''
        }

      }
    }

  }
}