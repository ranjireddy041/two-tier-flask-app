pipeline {
    agent any
	
	tools {
        sonarQubeScanner 'sonar-scanner'
    }

    stages {
        stage('checkout') {
            steps {
                echo 'code checkout from github'
				git branch: 'master', url: 'https://github.com/ranjireddy041/two-tier-flask-app.git
            }
        }
		
		 stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies'
                sh '''
                python3 -m venv venv
                source venv/bin/activate
                pip install -r requirements.txt
                '''
            }
		}	
		stage('SonarQube Scan') {
    steps {
        withSonarQubeEnv('sonarqube-server') {
            sh '''
            sonar-scanner \
            -Dsonar.projectKey=flaskapp \
            -Dsonar.sources=. \
            -Dsonar.host.url=http://54.85.108.8:9000 \
            -Dsonar.login=squ_a51dc249be8695b08004eb7c1ad2c3f058a7f470
            '''
        }
    }
}	
		stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -t ranjidockerhub/flaskapp:${BUILD_NUMBER} .'
            }
        }
		stage('push to docker hubrepo'){
			steps{
				echo 'push to docker hub repo'
				script {
					withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
					sh ' docker login -u ranjidokcerhub -p ${dockerhub} '
					}
					sh ' docker push ranjidokcerhub/flaskapp:-${BUILD_NUMBER} '
					 echo 'pushed to dockerhub'
				}	
			}
		}
		stage('update deployment file in github'){
			
				environment {
					GIT_REPO_NAME = "two-tier-flask-app"
					GIT_USER_NAME = "ranjireddy041"
				}	
				steps{
					echo ' update deployment file'
					withCredentials([string(credentialsId: 'github', variable: 'github')]) {
					sh ''' git config user.email " ranji.reddy@gmail.com"
							 git config user.name "ranjeeth"
							 BUILD_NUMBER=${BUILD_NUMBER}
							 sed -i "s/flaskapp:.*/flaskapp:${BUILD_NUMBER}/g" deploymentfile/deployment.yaml
							 git add .
							 git commit -m "update flaskapp deployment image to the version ${BUILD_NUMBER}"
							 git push https://${github}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD: main '''
							 
					}	
				}
		}
	}
}
