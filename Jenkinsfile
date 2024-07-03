def registry = 'https://varedla.jfrog.io/'

pipeline {
    tools {
        maven "Maven3"
    }
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'main', url: 'https://github.com/iam-aredla/springbootapp.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Unit Test') {
            steps {
                echo '<----------------------Unit Test Under Progress-------------------->'
                sh 'mvn surefire-report:report'
                echo '<----------------------Unit Test Finished------------------------->'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=springbootapp -Dsonar.projectKey=iam-aredla_springbootapp'''
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer(url: registry + "/artifactory", credentialsId: "jfrogaccess")
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/springbootApp.jar",
                                "target": "ncpl-libs-release",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ecrrep01 .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 851725351538.dkr.ecr.us-east-1.amazonaws.com'
                    sh 'docker tag ecrrep01:latest 851725351538.dkr.ecr.us-east-1.amazonaws.com/ecrrep01:latest'
                    sh 'docker push 851725351538.dkr.ecr.us-east-1.amazonaws.com/ecrrep01:latest'
                }
            }
        }
    }
}
