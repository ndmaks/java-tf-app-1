#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        maven 'Maven'
        terraform 'Terraform'
    }
    environment {
        IMAGE_NAME = "nedimaksu/java-app:java-maven-${BUILD_NUMBER}"
    }
    stages {
    
        /* stage('test') {
            steps {
                script {
                    echo "test the application"
                    sh 'mvn test'
                }
            }
        } */
 /*        stage('build jar') {
            steps {
                script {
                    echo "building the application jar"
                    sh 'mvn package'
                }
            }
        }
 */
        stage('build image') {
            steps {
                script {
                    echo "building the docker image"
                     withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'docker build -t ${IMAGE_NAME} .'
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh 'docker push ${IMAGE_NAME}'
                    }
                }
            }
        }
        stage('provision server') {
           steps {
                script {
                    dir('Terraform') {
                        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',credentialsId: "AWS-ID",accessKeyVariable: 'AWS_ACCESS_KEY_ID',secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                            sh "terraform init"
                            sh "terraform apply --auto-approve"
                            sh "terraform destroy -auto-approve"
                            EC2_PUBLIC_IP = sh(script: "terraform output ec2_public_ip",returnStdout: true).trim() 
                        }
                    }
                }
            }
        }
    }
}
