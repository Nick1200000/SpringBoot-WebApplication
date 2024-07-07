# SpringBoot-WebApplication CI/CD Pipeline

This repository contains the CI/CD pipeline for a Spring Boot web application. The pipeline automates the process of building, testing, analyzing, and deploying the application using Jenkins, Maven, SonarQube, Docker, and Trivy.

## Prerequisites

Before you begin, ensure you have the following installed:

1. **Jenkins** with the following plugins:
   - Git Plugin
   - Maven Integration Plugin
   - SonarQube Scanner Plugin
   - OWASP Dependency-Check Plugin
   - Docker Pipeline Plugin
   - Trivy Plugin

2. **Tools**:
   - Java Development Kit (JDK 17)
   - Apache Maven (version 3.x)
   - SonarQube
   - Docker
   - Trivy

## Pipeline Overview

The Jenkins pipeline includes the following stages:

1. **Git Checkout**: Checks out the source code from the GitHub repository.
2. **Code Compile**: Compiles the code using Maven.
3. **Run Test Cases**: Executes the test cases using Maven.
4. **SonarQube Analysis**: Performs static code analysis using SonarQube.
5. **OWASP Dependency Check**: Scans for known vulnerabilities in dependencies.
6. **Maven Build**: Builds the application using Maven.
7. **Docker Build & Push**: Builds a Docker image and pushes it to Docker Hub.
8. **Docker Image Scan**: Scans the Docker image for vulnerabilities using Trivy.

## Pipeline Script

Here is the Jenkins pipeline script:

```groovy
pipeline {
    agent any
    
    tools {
        jdk 'jdk17' // Updated to use Java 17
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/SpringBoot-WebApplication.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Run Test Cases') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Java-WebApp \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Java-WebApp
                    '''
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Maven Build') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t webapp ."
                        sh "docker tag webapp nikhil052/webapp:latest"
                        sh "docker push nikhil052/webapp:latest"
                    }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image nikhil052/webapp:latest"
            }
        }
    }
}
```
