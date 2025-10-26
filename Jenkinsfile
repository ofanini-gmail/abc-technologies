pipeline {
    agent any
    stages {
        stage('Code Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    userRemoteConfigs: [[
                        url: 'https://github.com/ofanini-gmail/abc-technologies.git',
                        credentialsId: 'gitaccess'
                    ]],
                    branches: [[name: '**']]
                ])
            }
        }

        stage('Code Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    cp /var/lib/jenkins/workspace/$JOB_NAME/target/*.war abc.war
                    docker build -t abc_tech:$BUILD_NUMBER .
                    docker tag abc_tech:$BUILD_NUMBER ofanini/abc_tech:$BUILD_NUMBER
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_creds',
                                                  usernameVariable: 'DOCKERHUB_USER',
                                                  passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                      echo "Logging in to Docker Hub..."
                      echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                      docker push ofanini/abc_tech:$BUILD_NUMBER
                      docker logout
                    '''
                }
            }
        }

	stage('Deploy as container') {
	    steps {
	        sh '''
	          echo "Stopping old container if running..."
	          docker ps -q | xargs -r docker stop || true
	          docker ps -aq | xargs -r docker rm || true

	          echo "Starting container on fixed port 8080..."
	          docker run -d --name abc_container -p 80:8080 ofanini/abc_tech:$BUILD_NUMBER
	          echo "Currently running containers:"
	          docker ps

	        '''
	    }
	}

    }
}

