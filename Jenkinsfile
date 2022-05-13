pipeline {
  environment {
    registry = 'ajduet/project2'
    dockerHubCreds = 'docker_hub'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Quality Gate') {
        steps{
            echo 'Quality Gate'
        }
    }
    stage('Unit Testing') {
        when {
            anyOf {branch 'ft_*'; branch 'bg_*'}
        }
        steps{
            withMaven {
                sh 'mvn test'
            }

            junit skipPublishingChecks: true, testResults: 'target/surefire-reports/*.xml'
        }
    }
    stage('Build') {
        when {
            branch 'main'
        }
        steps{
            withMaven {
                sh 'mvn package -DskipTests'
            }
        }
    }
    stage('Docker Image') {
        when {
            branch 'main'
        }
        steps{
            script {
                echo "$registry:$currentBuild.number"
                dockerImage = docker.build "$registry:$currentBuild.number"
            }
        }
    }
    stage('Docker Deliver') {
        when {
            branch 'main'
        }
        steps{
            script {
                docker.withRegistry("", dockerHubCreds) {
                    dockerImage.push("$currentBuild.number")
                    dockerImage.push("latest")
                }
            }
        }
    }
    stage('Wait for approval') {
        when {
            branch 'main'
        }
        steps {
            script {
            try {
                timeout(time: 1, unit: 'MINUTES') {
                    approved = input message: 'Deploy to production?', ok: 'Continue',
                        parameters: [choice(name: 'approved', choices: 'Yes\nNo', description: 'Deploy build to production')]

                    if(approved != 'Yes') {
                        error('Build did not pass approval')
                    }
                }
            } catch(error) {
                error('Build failed because timeout was exceeded');
            }
        }
        }
    }
    stage('Deploy'){
        when {
            branch 'main'
        }
        steps{
            sh 'sed -i "s/%TAG%/$BUILD_NUMBER/g" ./k8s/project2.deployment.yaml'
            step([$class: 'KubernetesEngineBuilder',
                projectId: 'devopssre-220404',
                clusterName: 'devopssre-220404-gke',
                zone: 'us-east1',
                manifestPattern: 'k8s/',
                credentials: 'devopssre-220404'
                verifyDeployments: true
            ])

            cleanWs();
        }
    }
  }
}