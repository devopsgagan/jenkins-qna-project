pipeline {
    agent any

    environment {
        PATH = "$PATH:/opt/apache-maven-3.9.6/bin"
    }

    stages {
        stage('Setup parameters') {
            steps {
                script {
                    properties([
                        parameters([
                            choice(
                                choices: ['DEV', 'QA'],
                                name: 'TOMCAT_ENV_NAME'
                            ),                           
                        ])
                    ])
                   
                }
            }
        }
        stage('git checkout') {
            steps {
                git branch: 'master', credentialsId: 'devopsgagan', url: 'https://github.com/devopsgagan/jenkins-freestyle.git'
            }
        }
        stage('maven clean and build'){
            steps {
                sh 'mvn clean package'
            }
        }
        stage('sonarqube analysis'){
            steps {
                withSonarQubeEnv('sonarqube8.9.2') { 
                sh "mvn sonar:sonar"
                }
            }
        }
        stage('Upload to Nexus Repository') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'student-task-tracker', classifier: '', file: 'target/student-task-tracker.war', type: 'war']], credentialsId: 'nexus-repo', groupId: 'com.example', nexusUrl: '3.81.97.220:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0.0-SNAPSHOT'
            }
        }
        stage('Approval & Deploy to DEV-tomcat') {
            when {
                expression {
                    return params.TOMCAT_ENV_NAME == 'DEV'
                }
            }
            steps {
                timeout(time: 120, unit: 'MINUTES') {
                    input (message: "Deploy to QA?")
                }
                deploy adapters: [tomcat9(credentialsId: 'dev-tomcat', path: '', url: 'http://34.224.86.66:8080/')], contextPath: null, war: '**/*.war'
            }
        }
         stage('Approval & Deploy to QA-tomcat') {
            when {
                expression {
                    return params.TOMCAT_ENV_NAME == 'QA'
                }
            }
            steps {
                timeout(time: 120, unit: 'MINUTES') {
                    input (message: "Deploy to DEV?")
                }
                deploy adapters: [tomcat9(credentialsId: 'qa-tomcat', path: '', url: 'http://18.207.167.221:8080/')], contextPath: null, war: '**/*.war'
            }
        } 
    }
    post {
        always {
            emailext body: '$DEFAULT_CONTENT', subject: '$DEFAULT_SUBJECT', to: '$DEFAULT_REPLYTO'
        }
    }
}
