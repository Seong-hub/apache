pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: lachlanevenson/k8s-kubectl
    command:
    - cat
    tty: true
  - name: docker
    image: docker:23.0
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
            '''
        }
    }
    stages {
        stage("Get Source") {
            steps {
                git branch: 'main', url: 'https://github.com/Seong-hub/apache.git'
            }
        }


	stage("Build Microservice image") {
		steps {                 
			container("docker") {
				script {
					try {
					appImage = docker.build("lalll5555/apache")
					} catch (e) {sh "echo docker build fail"}
						docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-ID') {
						try {
					               appImage.push("${env.BUILD_NUMBER}")
				                       appImage.push("latest")
						} catch (e) { sh 'echo docker push fail'}
					}
                              	}
			}                                                                                
              }
      	}
	stage( "depolyment list check" ) {
		steps {
			container("kubectl") {
                		sh "kubectl get deployments.apps -n apache "
                	}
		}       
	}
        stage( "Clean Up Existing Deployments,Service,Ingress" ) {
       		steps {
		        container("kubectl") {
				script {
					try { 
			       	        	sh "kubectl delete -n apache deployments.apps apache-depolyment"
			       	        	sh "kubectl delete -n apache services apache-service"
						sh "kubectl delete -n apache ingress apache-ingress"
			       	       	} catch (e) { sh "echo Deployments,Service,Ingress not delete (hint : kubectl get deployments,service,ingress -n apache)" }
				}
			}
                }
        }
	stage( "Deploy to Cluster" ) {
		steps {
        	        container("kubectl") {
//                	        sh "kubectl run apache --image lalll5555/apache:BUILD_NUMBER -n apache"
                	        sh "kubectl apply -f ./apache_deployment.yaml -n apache"
	                        sh "sleep 5"
			}
        	}
        }
	stage( "Check Deployments,Service,Ingress" ) {
		steps {
        	        container("kubectl") {
				sh "kubectl get deployment.apps -n apache"
				sh "kubectl get services -n apache"
				sh "kubectl get ingress -n apache"
			}
        	}
        }

}
}
