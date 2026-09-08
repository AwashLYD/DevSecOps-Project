pipeline {
    agent any

    tools {
        jdk 'jdk26'
        nodejs 'node26'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/AwashLYD/DevSecOps-Project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate(
                        abortPipeline: false,
                        credentialsId: 'Sonar-token'
                    )
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

stage('OWASP FS SCAN') {
    steps {
        withEnv([
            "PATH+JDK=${tool 'jdk26'}/bin",
            "PATH+NODE=${tool 'node26'}/bin"
        ]) {
            withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_KEY')]) {
                // MISSING STEP: You must invoke the scan here before publishing
                dependencyCheck(
                    additionalArguments: "--scan ./ --format XML --format HTML --nvdApiKey ${env.NVD_KEY}",
                    odcInstallation: 'OWASP-DC'
                )
            }
        }
    }
    post {
        always {
            dependencyCheckPublisher allowMissingFiles: true, pattern: '**/dependency-check-report.xml'
        }
    }
}       
       stage('TRIVY FS SCAN') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: 'docker',
                        toolName: 'docker'
                    ) {
                        sh '''
                            docker build \
                            --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY \
                            -t netflix .
                        '''

                        sh 'docker tag netflix nasi101/netflix:latest'

                        sh 'docker push nasi101/netflix:latest'
                    }
                }
            }
        }

        stage('TRIVY IMAGE SCAN') {
            steps {
                sh 'trivy image nasi101/netflix:latest > trivyimage.txt'
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                    docker rm -f netflix || true

                    docker run -d \
                    --name netflix \
                    -p 8081:80 \
                    nasi101/netflix:latest
                '''
            }
        }
    }
}
