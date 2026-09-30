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
    }
}