pipeline {
  agent any
  tools { 
        maven 'maven_3.5.2'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
			   sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=newbuggywebapp -Dsonar.organization=buggywebapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=8ea8762ea3bef1899ddac2d1ac6d059953b7e2a9'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build') { 
            steps { 
			   script{
				   sh "docker build -t secdevops1 ."
				}
			}
    }
           

	stage('ECRpush') { 
            steps { 
			   script{
				   sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 297621708399.dkr.ecr.ap-south-1.amazonaws.com"
				   sh  "docker tag secdevops1:latest 297621708399.dkr.ecr.ap-south-1.amazonaws.com/secdevops1:latest"
				   sh  "docker push 297621708399.dkr.ecr.ap-south-1.amazonaws.com/secdevops1:latest"
				}
			}
    }
	
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}
	
	}
}
