pipeline {
    agent any
    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN_1') // Binds the SONAR_TOKEN credential to an environment variable
        DOCKER_TOKEN = credentials('DOCKER_TOKEN')
    }
    tools {
        nodejs 'NodeJS' // Match the name from Global Tool Configuration
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mivo1101/SIT753-7.3HD.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t govietnam-website:latest .'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm test || true'
            }
        }
        stage('SonarCloud Code Quality Analysis') {
            steps {
                sh '''
                    # Download and extract SonarScanner CLI
                    curl -o sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.1.0.4889.zip
                    unzip -o sonar-scanner.zip
                    rm -rf sonar-scanner || true
                    mv sonar-scanner-7.1.0.4889 sonar-scanner
                    ./sonar-scanner/bin/sonar-scanner \
                        -Dsonar.projectKey=mivo1101_SIT753-7.3HD \
                        -Dsonar.organization=mivo1101 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
        stage('Snyk Security Scan') {
            steps {
                echo 'Running Snyk Dependency Scan...'
                snykSecurity(
                    snykInstallation: 'Snyk',
                    snykTokenId: 'SNYK_TOKEN_1',
                    organisation: 'mivo1101',
                    projectName: 'mivo1101_SIT753-7.3HD',
                    severity: 'low',
                    additionalArguments: '--json --severity-threshold=low --file=package.json'
                )
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Use docker-compose to deploy
                    sh 'docker-compose -f docker-compose.yml down || true'
                    sh 'docker-compose -f docker-compose.yml up -d'
                }
            }
        }
        stage('Release') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_TOKEN', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker tag govietnam-website:latest $DOCKER_USER/govietnam-website:latest'
                        sh 'docker push $DOCKER_USER/govietnam-website:latest'
                    }
                }
            }
        }
    }
}
