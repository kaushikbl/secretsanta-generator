pipeline {
    
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3.6.0'
    
    }
    environment{
        SCANNER_HOME = tool 'sonar-server'
        
    }
    
    stages {
        stage('Git-checkout') {
            steps {
                git credentialsId: 'git-jenkins', url: 'https://github.com/kaushikbl/secretsanta-generator.git'
            }
        }
    
        stage('compile') {
            steps {
                sh "mvn clean compile"
            }
        }    
        
        //stage('OWASP-scan') {
        //    steps {
        //        dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
        //        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //    }
        //}
        
        stage('sonarqube') {
            steps {
             withSonarQubeEnv('sonarqube-server') {
             sh ''' ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=Santa -Dsonar.projectKey=Santa -Dsonar.java.binaries=. '''
            }
        }
        }
        
        stage('mvn-Build') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage("Build Docker Image") { 
            steps {
                script {
                    readpom = readMavenPom file: '';
                    
                    artifactversion = readpom.version;
                    
                    timeStamp = Calendar.getInstance().getTime().format("dd-MM-yyyy.HH-mm-ss",TimeZone.getTimeZone('IST'))
                    
                    buildNumber = "${BUILD_NUMBER}"
                    
                    version_number = artifactversion + "." + buildNumber + "." + timeStamp + "."
                    
                    currentBuild.displayName = version_number
                    
                    echo version_number
                    
                    
                     sh "docker build -t kaushikbl/secretsanta:${version_number} ."
                }
           }
        }
        
        stage('Docker Image Scan') {
            steps {
               sh "trivy image kaushikbl/secretsanta:${version_number}"
            }
        }
        
        stage("Login & push to DockerHub") {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker-jenkins', toolName: 'docker') {
                   sh "docker push kaushikbl/secretsanta:${version_number}"
                }
            }
        }
    }
        stage('Deploy app') { 
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-jenkins', toolName: 'docker') {
                    sh "docker run -d --name secretsanta -p 8081:8080 kaushikbl/secretsanta:${version_number}"
                }
            }
        }
    }
}
}