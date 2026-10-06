pipeline {

    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
        skipDefaultCheckout(true)
    }

    tools {
        maven "maven-3.9.16"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                notifyBuild('STARTED')
                git branch: 'dev',
                    url: 'https://github.com/shashikiran05/maven-webapplication-project-kkfunda.git'
            }
        }

        stage('Compile') {
            steps {
                echo 'Compiling application...'

                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'

                sh 'mvn test'
            }

            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building WAR file...'

                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Verify Artifact') {
            steps {
                echo 'Checking generated WAR file...'

                sh '''
                    echo "Contents of target directory:"
                    ls -lh target/

                    echo "Generated WAR files:"
                    find target -name "*.war" -type f

                    if ! ls target/*.war >/dev/null 2>&1; then
                        echo "ERROR: WAR file was not generated!"
                        exit 1
                    fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                sh 'mvn sonar:sonar'
            }
        }
        stage('Deploy to Nexus') {
            steps {
                echo 'Deploying artifact to Nexus...'
                sh 'mvn deploy -DskipTests'
             }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archiving WAR file in Jenkins...'

                archiveArtifacts artifacts: 'target/*.war',
                                 fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR file to Tomcat...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'tomcat-credentials',
                        usernameVariable: 'TOMCAT_USER',
                        passwordVariable: 'TOMCAT_PASSWORD'
                    )
                ]) {

                    sh '''
                        curl --fail --show-error --silent \
                        -u "$TOMCAT_USER:$TOMCAT_PASSWORD" \
                        --upload-file target/*.war \
                        "http://13.235.82.75:8080/manager/text/deploy?path=/maven-web-application&update=true"
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying Tomcat deployment...'

                sh '''
                    curl --fail --show-error \
                    "http://13.235.82.75:8080/maven-web-application/"
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'PIPELINE COMPLETED SUCCESSFULLY'
            echo 'WAR deployed successfully to Tomcat'
            echo '======================================'

            script {
                notifyBuild('SUCCESS')
            }
        }

        failure {
            echo '======================================'
            echo 'PIPELINE FAILED'
            echo 'Check the failed stage in Jenkins'
            echo '======================================'

            script {
                notifyBuild('FAILURE')
            }
        }

        always {
            echo 'Cleaning Jenkins workspace...'

            cleanWs()
        }
    }
}


// Notification method
def notifyBuild(String buildStatus = 'STARTED') {

    buildStatus = buildStatus ?: 'SUCCESS'

    def colorCode

    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    switch (buildStatus) {

        case 'STARTED':
            colorCode = '#FFFF00'
            break

        case 'SUCCESS':
            colorCode = '#00FF00'
            break

        default:
            colorCode = '#FF0000'
    }

    slackSend(
        color: colorCode,
        message: summary,
        channel: '#shashib10'
    )
}
