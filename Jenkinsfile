pipeline {
	agent any
  environment {
        VERSION = 'latest'
        PROJECT = 'capstone-dragonboat-app'
				IMAGE = "$PROJECT"
				ECRURL = "https://712123673000.dkr.ecr.us-east-1.amazonaws.com/$PROJECT"
				ECRURI = "712123673000.dkr.ecr.us-east-1.amazonaws.com/$PROJECT"
				ECRCRED = 'ecr:us-east-1:dragonboat-ecr-cred'
  }
	stages {
		stage("Lint Dockerfile") {
			steps {
				sh "docker run --rm -i hadolint/hadolint:v1.18.0 < Dockerfile"
			}
		}
		stage('Build preparations') {
      steps {
        script {
            // calculate GIT lastest commit short-hash
            gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            shortCommitHash = gitCommitHash.take(7)
            // calculate a sample version tag
            VERSION = shortCommitHash
            // set the build display name
            currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
        }
      }
		}
		stage('Docker build') {
		  steps {
			  script {
			    // Build the docker image using a Dockerfile
			    docker.build("$IMAGE")
			  }
      }
		}
		stage('Docker push') {
      steps {
        script {
          // Push the Docker image to ECR
	    docker.withRegistry(ECRURL, ECRCRED) {
						docker.image(IMAGE).push("latest")
            docker.image(IMAGE).push(VERSION)
          }
			  }
      }
		}
    stage('Kubernetes Deploy') {
      steps {
				withAWS(credentials: 'aws-cred', region: 'us-east-1') {
					sh "aws eks --region us-east-1 update-kubeconfig --name UdacityCapStone-Cluster"
                    // sh "kubectl apply -f k8s/aws-auth-cm.yaml"
                    // sh "kubectl set image deployments/$PROJECT $PROJECT=$ECRURI:$VERSION"
					
					// sh "kubectl apply -f k8s/deployment.yml"
					// sh "kubectl apply -f k8s/service.yml"
					// sh "kubectl get nodes"
                    // sh "kubectl get pods"
				}
      }
    }
  }
	post {
		always {
		    // make sure that the Docker image is removed
		    sh "docker rmi $IMAGE | true"
		}
	}
}