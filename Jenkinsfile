def getBranchName() {
	return scm.branches[0].name
}
pipeline
{   
  agent any 
  environment 
  {
	registry = "rohit2522/nagp-jenkins-assignment"
	  branch = getBranchName()
  }
  tools
  {
	 maven 'Maven3'
  }     
  options
  {
	  skipDefaultCheckout()
	  disableConcurrentBuilds()
  }
  stages
  {
	 stage ('Checkout')
	 {
		steps
		{
		   echo "build in master branch - 1"
		   checkout scm
		   
		}
	 }
	 stage ('Build')
	 {
		steps
		{
		   echo "build in master branch - 2"
		   bat "mvn clean install -Dhttps.protocols=TLSv1.2"
		}
	 }
	 stage ('Unit Testing')
	 {
		 when {
			expression {
				branch == 'master'
			}
		}

		steps
		{
		   bat "mvn test"
		}
	 }
	 stage ('Sonar Analysis')
	 {
		when {
			expression {
				branch == 'develop'
			}
		}

		steps
		{
		   withSonarQubeEnv("Test_Sonar")
		   {
			  bat "mvn sonar:sonar"
		   }
		}

	 }
	stage('Docker image') 
	{
		  steps
		  {
			  script {
				  if (branch == 'master') {
					bat "docker build -t i-rohit2522-master:${BUILD_NUMBER} --no-cache -f Dockerfile ."   
				  } else { 
					bat "docker build -t i-rohit2522-develop:${BUILD_NUMBER} --no-cache -f Dockerfile ."   
				  }
			  }
		  }
	}
	stage('Container') {
		parallel{
			stage('Precontainer Check') {
				steps {
					script {
						if (branch == 'master') {
							bat "docker rm -f c-rohit2522-master || exit 0 && docker rm c-rohit2522-master || exit 0"
						} else {
							bat "docker rm -f c-rohit2522-develop || exit 0 && docker rm c-rohit2522-develop || exit 0"
						}
					}
				}
			}
			stage('Push to Dockerhub Repo') {
			  steps{
				  script {
					  if(branch == 'master') {
						 bat "docker tag i-rohit2522-master:${BUILD_NUMBER} ${registry}:master-${BUILD_NUMBER}"
						 bat "docker tag i-rohit2522-master:${BUILD_NUMBER} ${registry}:master-latest"
						 withDockerRegistry([credentialsId: 'Test_Docker', url:""]){
							bat "docker push ${registry}:master-${BUILD_NUMBER}"
							bat "docker push ${registry}:master-latest"
						 }
					  } else {
						  bat "docker tag i-rohit2522-develop:${BUILD_NUMBER} ${registry}:develop-${BUILD_NUMBER}"
						 bat "docker tag i-rohit2522-develop:${BUILD_NUMBER} ${registry}:develop-latest"
						 withDockerRegistry([credentialsId: 'Test_Docker', url:""]){
							bat "docker push ${registry}:develop-${BUILD_NUMBER}"
							bat "docker push ${registry}:develop-latest"
						 }
					   }
				  }
						 
							
			}
			}
		}
	}
	stage('Docker deployment') 
	{            
		 steps{
			 script {
				 if(branch === 'master') {
					bat "docker run --name c-rohit2522-master -d -p 7200:8080 ${registry}:master-latest" 
				 } else {
					bat "docker run --name c-rohit2522-develop -d -p 7300:8080 ${registry}:develop-latest" 
				 }
			 }
		 }
	}
  }
}
