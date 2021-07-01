pipeline {
    agent any
    tools {
        jdk "jdk11"
        maven "mvn"
    }
    stages {

       stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Maven Tests') {
            steps {
                sh 'mvn clean test'
                junit '**/surefire-reports/*Test.xml'
            }
        }

        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=demo'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                sleep(10);
                waitForQualityGate abortPipeline: true;
            }
        }

        stage('Maven Install') {
            steps {
                sh 'mvn clean install -Dmaven.test.skip=true'
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t yeskay16/app:${env.BUILD_NUMBER} -f Dockerfile ."
            }
        }
	
        stage('Publish Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'sk_dockerhub_creds', passwordVariable: 'PASS', usernameVariable: 'USER')])  {
                    sh 'docker login --username ${USER} --password ${PASS}'
                    sh "docker push yeskay16/app:${env.BUILD_NUMBER}"
                }
            }
        }

    }
}
