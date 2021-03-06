pipeline{
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
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome }/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=5edf6c333113ae95a2b55de3651f3c946bf239ed -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                } 
            }
        }
        stage('Quality Gate'){
            steps {
                sleep(20) 
                timeout(time: 1, unit: 'MINUTES'){
                     waitForQualityGate abortPipeline: true 
                }
            }
        }
        stage('Deploy Backend'){
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'

            }
        }
        stage('API Test'){
            steps {
                dir('api-test'){
                git credentialsId: 'TomcatLogin', url: 'https://github.com/denisefaggionato-28/tasks-api-test'
                bat 'mvn test'
                }
            }
        }
        stage('Deploy FrontEnd'){
            steps {
                dir('frontend'){
                git credentialsId: 'TomcatLogin', url: 'https://github.com/denisefaggionato-28/tasks-frontend'  
                 bat 'mvn clean package'
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage('Functional Test'){
            steps {
                dir('Functional-test'){
                git credentialsId: 'TomcatLogin', url: 'https://github.com/denisefaggionato-28/Test-Funcional'
                bat 'mvn test'
                }
            }
        }
        stage('Deploy Prod'){
            steps {
               bat 'docker-compose build'
               bat 'docker-compose up -d'  
            }
       }
       stage('Health Check'){
            steps {
                sleep(20)
                dir('Functional-test'){
                bat 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post {
        always{
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, Functional-test/target/surefire-reports/*.xml, Functional-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', onlyIfSuccessful: true
        }
        unsuccessful{
            emailext body: 'see the attached log below', compressLog: true, subject: 'Build $BUILD_NUMBER has falied', to: 'denisejonata@gmail.com'
        }
        fixed{
            emailext body: 'see the attached log below', compressLog: true, subject: 'Build is fine!!', to: 'denisejonata@gmail.com'
        }  

    }
}
