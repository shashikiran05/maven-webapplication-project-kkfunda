
pipeline 
{
  agent any
  tools
  {
    maven "maven-3.9.16"
  }
  stages
  {
    stage('git checkout')
    {
      steps
      {
      notifyBuild('STARTED')
      git branch: 'master', url: 'https://github.com/shashikiran05/maven-webapplication-project-kkfunda.git'
      }
    }
    stage('COMPILE')
    {
      steps
      {
        sh "mvn compile"
      }
    }
   stage('Build')
    {
      steps
      {
        sh "mvn clean package"
      }
    }
    stage('SQ Report')
    {
      steps
      {
        sh "mvn sonar:sonar"
      }
    }
   stage('Articat Backup')
    {
      steps
      {
        sh "mvn deploy"
      }
    }
   stage('Deploy To Tomcat')
    {
      steps
      {
        sh """

      curl -u kk:password \
--upload-file /var/lib/jenkins/workspace/Declarative-PL-webapp/target/maven-web-application.war \
"http://3.109.207.137:8080/manager/text/deploy?path=/maven-web-application&update=true"
          
        """
      }
    }
    stage('airtel-dev')
    {
      steps
      {
         build job: 'airtel-dev'
      }
    }



  } //stages ending

post {
  success {

    script
    {
     notifyBuild(currentBuild.result)
    }
    
  }
  failure {

  script
  {
    notifyBuild(currentBuild.result)

  }
   
  }
}
    
} //pipeline ending

// Notification method
def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus = buildStatus ?: 'SUCCESS'

    def colorCode
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    switch (buildStatus) {
        case 'STARTED':
            colorCode = '#FFFF00' // Yellow
            break
        case 'SUCCESS':
            colorCode = '#00FF00' // Green
            break
        default:
            colorCode = '#FF0000' // Red
    }

    slackSend(color: colorCode, message: summary, channel: '#airtel-project')
}


