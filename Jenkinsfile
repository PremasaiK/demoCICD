
pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('docker-premasaik')
		POD_NAME = "tmp"
	}
  
	stages {
    
		stage('Build') {

			steps {
				echo "${BUILD_NUMBER}"
				sh 'docker build  --no-cache -t premasaik/kubernetes-101:${BUILD_NUMBER} .'
			}
		}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push') {

			steps {
				sh 'docker push premasaik/kubernetes-101:${BUILD_NUMBER}'
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
							sh 'ssh premasai@127.0.0.1 sed -i "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" /home/premasai/deployment.yaml'
							sh 'ssh premasai@127.0.0.1 cat /home/premasai/deployment.yaml'
							sh 'ssh premasai@127.0.0.1 kubectl apply -f /home/premasai/deployment.yaml'
							sleep 5
							sh 'ssh premasai@127.0.0.1 kubectl apply -f service.yaml'
							sleep 5
							ret0 = sh ( script:'ssh premasai@127.0.0.1 kubectl get pods | grep  "kubernetes-101.*Running" | awk \'{print $1}\'',returnStdout: true).trim()
							println ret0
							ret1 = sh ( script:'ssh premasai@127.0.0.1 kubectl get pods | grep  "kubernetes-101.*Running" | awk \'{print $3}\'',returnStdout: true).trim()
							println ret1
							ret2 = sh ( script:'ssh premasai@127.0.0.1 kubectl get pods | grep  kubernetes-101-* |  sed -n 2p | awk \'{print $1}\'',returnStdout: true).trim()
							println "${ret2}"	
						        if (ret1 == "Running") {
								      echo "execute portforward command on cosole: kubectl port-forward  ${ret0} 30008:3000"        
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
