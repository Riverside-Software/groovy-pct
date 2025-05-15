// Syntax check with this command line
// curl -k -X POST -F "jenkinsfile=<Jenkinsfile" https://ci.rssw.eu/pipeline-model-converter/validate

pipeline {
  agent none
  options {
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 30, unit: 'MINUTES')
    skipDefaultCheckout()
  }

  stages {
    stage('Maven Central') {
      agent { label 'Linux-Office03' }
      environment {
        MAVEN_GPG_PASSPHRASE=credentials('GPG_KEY')
      }
      steps {
        checkout([$class: 'GitSCM', branches: scm.branches, extensions: scm.extensions + [[$class: 'CleanCheckout']], userRemoteConfigs: scm.userRemoteConfigs])
        script {
          def jdk = tool name: 'JDK17', type: 'jdk'
          def mvn = tool name: 'Maven 3', type: 'maven'

          withEnv(["JAVA_HOME=${jdk}"]) {
            sh "${mvn}/bin/mvn -P release clean package verify deploy -Dgit.commit=\$(git rev-parse --short HEAD)"
            mail body: "https://central.sonatype.com/publishing/deployments", to: "g.querret@riverside-software.fr", subject: "groovy-pct - Publish artifact on Central"
          }
        }
      }
    }
  }
}
