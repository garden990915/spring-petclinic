pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'M3'
    }
    environment { 
        DOCKERHUB_CREDENTIALS = credentials('dockerCredentials')
        K8S_TOKEN = credentials('k8sCredentials')
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/garden990915/spring-petclinic.git',
                branch: 'main', credentialsId: 'gitToken'
            }
            post {
                success {
                    echo 'Success git clone step'
                }
                failure {
                    echo 'Fail git clone step'
                }
            }
        }
        
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true package'
            }
            post {
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        
        stage('Docker Image Build') {
            steps {
                echo 'Docker Image build'                
                dir("${env.WORKSPACE}") {
                    sh """
                    docker build -t goldengarden/spring-petclinic:$BUILD_NUMBER .
                    docker tag goldengarden/spring-petclinic:$BUILD_NUMBER goldengarden/spring-petclinic:latest
                    """
                }
            }
        }

        stage('Docker Login') {
            steps {
                // docker hub 로그인
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Docker Image Push') {
            steps {
                echo 'Docker Image Push'  
                sh "docker push goldengarden/spring-petclinic:latest"  // docker push
            }
        }
        stage('Cleaning up') { 
	    steps { 
              // docker image 제거
                echo 'Cleaning up unused Docker images on Jenkins server'
                sh """
                docker rmi goldengarden/spring-petclinic:$BUILD_NUMBER
                docker rmi goldengarden/spring-petclinic:latest
                """
            }
        }
        
        stage('Deploy to Kubernetes') {
              steps {
                echo 'Deploying to Kubernetes'
                script {
                        withKubeCredentials(kubectlCredentials: [[
                                caCertificate: '',
                                clusterName: 'Team1_k8s',
                                contextName: '',
                                credentialsId: 'k8sCredentials',
                                namespace: 'default', // 필요한 경우 네임스페이스 지정
                                serverUrl: 'https://10.100.202.114:6443'
                        ]]) {
	                    sh """
     	                    kubectl apply -f spring-petclinic-deployment.yml
                            kubectl apply -f spring-petclinic-ingress.yml
    	                    """
	                    sleep(15)
                            sh """
	                    kubectl rollout restart deployment/team1react -n team1
	                    kubectl rollout restart deployment/team1spring -n team1
	                    """
	                    sleep(10)
	                    sh """
     	                    kubectl get pods -n team1
	                    """  
                        }
                }
              }
        }
            
    }
}
