pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        GIT_CRED = 'github-cred'
        GIT_URL = 'git@github.com:goti03/mvn-project.git'
        SONARQUBE_URL = 'http://192.168.1.225:9000'
        SONARQUBE_TOKEN = credentials('sonarqube-token')
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {

        stage('Checkout Project...')
        {
            steps
                {
                    git branch: env.GIT_BRANCH,
                    credentialsId: env.GIT_CRED,
                    url: env.GIT_URL
                }
        }

        stage('Compile Code') {
            steps {
                script {
                    echo 'Compiling the source code...'
                    sh "mvn clean compile"
                }
            }
        }

        stage('Run Unit Tests & Generate Code Coverage') {
            steps {
                script {
                    echo 'Running Unit Tests and Generating Coverage Report...'
                    sh 'mvn test jacoco:report'
                }
            }
            post {
                always {
                    jacoco execPattern: '**/target/jacoco.exec', classPattern: '**/target/classes', sourcePattern: '**/src/main/java'
                }
            }
        }

        stage('Static Code Analysis with SonarQube') {
            steps {
                script {
                    echo 'Running Static Code Analysis using SonarQube...'
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn -X sonar:sonar'
                    }
                }
            }
        }

        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    echo 'Checking SonarQube Quality Gate...'
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

    }

    post {
        success {
            echo 'Build completed successfully!'
        }
        failure {
            echo 'Build failed! Check logs for details.'
        }
    }
}
