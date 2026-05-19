pipeline {
    agent any
    tools {
        maven 'Maven' // Must match the name in Jenkins Global Tool Config 
    }
    environment {
    JAVA_HOME = "/usr/lib/jvm/java-21-amazon-corretto.x86_64"
    PATH = "$JAVA_HOME/bin:$PATH"
}

    stages {
        stage('Git Checkout') {
            steps { checkout scm }
        }
         stage('Environment Check') {
            steps {
                // This will prove to you in the logs if it's actually using 21
                sh 'java -version'
                sh 'mvn -version'
            }
        }
        stage('Maven Build & Unit Test') {
            steps {
                sh 'mvn clean package' 
            }
        }
        stage('SonarQube Analysis') {
    steps {
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
            sh '''
            mvn sonar:sonar \
            -Dsonar.projectKey=devsecops-app \
            -Dsonar.host.url=http://3.84.67.2:9000 \
            -Dsonar.login=$SONAR_TOKEN
            '''
        }
    }
}
        stage('Security Scan (Dependency-Check)') {
            steps {
                dependencyCheck additionalArguments: "--scan . --nvdApiKey=4a9f5b3b-391e-4ecb-8e57-71ab56079f0f",
                                 odcInstallation: 'OWASP-DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Dependency Check') {
    steps {
        sh '''
        /opt/dc/bin/dependency-check.sh \
        --project demo \
        --scan . \
        --format HTML \
        --out report \
        --data /opt/dc/data \
        --noupdate || true
        '''
    }
}


        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t my-devops-app:latest .'
                // Optional: add push steps here if you have credentials set up
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image \
                  --scanners vuln \
                  --pkg-types os \
                  --severity HIGH,CRITICAL \
                  --no-progress \
                  my-devops-app:latest
                '''
            }
        }
        stage('Deploy to Container') {
            steps {
                sh 'docker rm -f my-running-app || true'
                sh 'docker run -d --name my-running-app -p 8081:8080 my-devops-app:latest'
            }
        }
        stage('Cleanup Trivy Cache') {
            steps {
                sh 'sudo rm -rf /var/lib/jenkins/.cache/trivy || true'
            }
        }
    }
    
        post {
            always {
                cleanWs()
            }
}

}
 
