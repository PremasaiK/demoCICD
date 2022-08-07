
pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('docker-premasaik')
		POD_NAME = "tmp"
	}
  
	stages {
    
		stage('Build') {

			steps {
				sh 'docker build -t premasaik/kubernetes-101:v3 .'
			}
		}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push') {

			steps {
				sh 'docker push premasaik/kubernetes-101:v3'
			}
		}
    
		stage('Deploy to K8s')
		{
			steps{
				     sshagent(['premasaik-k8s'])
				      {
					sh 'scp -v -r -o StrictHostKeyChecking=no /home/premasai/demoCICD/deployment.yaml /home/premasai/demoCICD/service.yaml premasai@127.0.0.1:/home/premasai'
					
					script{
						try{
							sh 'ssh premasai@127.0.0.1 kubectl apply -f deployment.yaml'
							sleep 5
							sh 'ssh premasai@127.0.0.1 kubectl apply -f service.yaml'
							sleep 5
							sh 'ssh premasai@127.0.0.1 kubectl get pods | grep kubernetes-101-*'
							sleep 5
							ret1 = sh ( script:'ssh premasai@127.0.0.1 kubectl get pods | grep kubernetes-101-* | awk \'{print $3}\'',returnStdout: true).trim()
							println ret1
							ret2 = sh ( script:'ssh premasai@127.0.0.1 kubectl get pods | grep kubernetes-101-* | awk \'{print $1}\'',returnStdout: true).trim()
							println "${ret2}"	
						        if (ret1 == "Running") {
								      echo "execute portforward command on cosole: kubectl port-forward  ${ret2} 30006:3000"        
	     							}
							     else {
								      println "${ret1}"
							         }
						      }catch(error)
							     {
							        sh 'issue with depyement.yaml/service.yaml'
							      }
					        }
				       } 
			      }
		     }
	    }

	post {
		always {
			sh 'docker logout '
		}
	}

}
