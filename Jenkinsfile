pipeline {
    agent any

    tools {
        jdk "jdk11"
        maven "mvn"
    }

    options {
        skipDefaultCheckout(true)
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

        stage('OWASP Scan and Publish') { 
            steps {
                dependencyCheck additionalArguments: 'dependency-check.sh --project "DevSecOps" --scan "/var/lib/jenkins/workspace/DevSecOps_CI_Pipeline/"', odcInstallation: 'OWASP_Dependency_Check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml', failedTotalCritical: 3, unstableTotalCritical: 1
                archiveArtifacts artifacts: 'dependency-check-report.*'
                script {
                    if (currentBuild.result == 'UNSTABLE') {
                        unstable('UNSTABLE: Dependency check')
                    } else if (currentBuild.result == 'FAILURE') {
                        error('FAILED: Dependency check')
                    }
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t yeskay16/devsecopsapp:${env.BUILD_NUMBER} -f Dockerfile ."
            }
        }
	
        stage('Publish Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'sk_dockerhub_creds', passwordVariable: 'PASS', usernameVariable: 'USER')])  {
                    sh 'docker login --username ${USER} --password ${PASS}'
                    sh "docker push yeskay16/devsecopsapp:${env.BUILD_NUMBER}"
                }
            }
        }

    }
}
