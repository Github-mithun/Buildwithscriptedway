# Buildwithscriptedway
node{
    
def mavenHome = tool name: 'maven3.9.6'

echo "The Job name is : ${env.JOB_NAME}"
echo "Jenkins Home dir is: ${env.JENKINS_HOME}"
echo "THe Jenkins node name is: ${env.NODE_NAME}"
echo "The Build number is: ${env.BUILD_NUMBER}"

properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '5', artifactNumToKeepStr: '5', daysToKeepStr: '5', numToKeepStr: '5')), [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false], [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])

stage('CheckoutCode'){
git branch: 'development', url: 'https://github.com/Github-mithun/maven-web-application.git'
}

stage('Build'){
sh "${mavenHome}/bin/mvn clean package"
}

stage('ExecuteSonarQubeReport'){
sh "${mavenHome}/bin/mvn clean sonar:sonar"
}

stage('UploadArtifactIntoNexus'){
sh "${mavenHome}/bin/mvn clean deploy"
}
stage('DeployAppIntoTomcat'){
sshagent(['68e217c5-e3a2-4aa8-b3a5-7d84b3549994']){
    sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@65.2.6.252:/opt/apache-tomcat-9.0.86/webapps"
}
}
}
