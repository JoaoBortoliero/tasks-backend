pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            tools { jdk 'JDK11' }
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=4a1a2db713304fe23c22b3525e4e3c1c25dfecb8 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn**,**/src/test**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(5)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend'){
            steps {
                deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: '/tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API test') {
            steps {
                dir('api-test') {
                    git branch: 'main', url: 'https://github.com/JoaoBortoliero/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend'){
            steps {
                dir('frontend') {
                    git branch: 'master', url: 'https://github.com/JoaoBortoliero/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: '/tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional test') {
            steps {
                dir('functional-test') {
                    git branch: 'main', url: 'https://github.com/JoaoBortoliero/tasks-functional-test'
                    bat 'mvn test'
                }
            }
        }
        stage ("Deploy Prod") {
            steps {
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
        stage ('Health Check') {
            steps {
                sleep(5)
                dir('functional-test') {
                    bat 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
}