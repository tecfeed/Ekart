pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
        
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'sk-dev', changelog: false, poll: false, url: 'https://github.com/tecfeed/Ekart.git'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'mvn test -Dmaven.test.skip=true'
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.url=http://192.168.31.200:9000/ -Dsonar.projectName=Ekart-Ulti \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=Ekart-Ulti'''
                }
            }
        }
        
        stage('OWASP Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DP'
	            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build Application') {
            steps {
                sh "mvn package -Dmaven.test.skip=true"
            }
        }
        stage('Deploy to Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -Dmaven.test.skip=true"
            }
            }
        }
        
        stage('Docker build image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '0e91734b-224b-4a19-bdad-41fc7c89d005', toolName: 'docker') {
                        sh "docker build -t ekartulti:1 -f docker/Dockerfile ."
                        
                    }
                }
        }        
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy --scanners vuln image ekartulti:1 > trivy-report.txt"
            }
        }
        stage('Docker tag & push image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '0e91734b-224b-4a19-bdad-41fc7c89d005', toolName: 'docker') {
                        sh "docker tag ekartulti:1 saurabh6870/ekartulti:1"
                        sh "docker push saurabh6870/ekartulti:1"
                    }
                }
        }        
        }deploy to kuber
	//
        stage('Kubernetes Deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-ulti-token', namespace: 'ekartapp', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.31.201:6443') {
                sh "kubectl apply -f deploymentservice.yml -n ekartapp"
                sh "kubectl get svc -n ekartapp"
            }
            }
        }
    }
}