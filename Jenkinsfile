def DOCKER_TAG
def icon = ":heavy_check_mark:"
/*
    Build a docker image
*/
def buildDockerImage(image_name, dockerfile, basedir) {
    echo "Building the demo-app Docker Image"
    sh "docker build -t ${image_name} -f ${basedir}/${dockerfile} ${basedir}/"
}
/*
    Publish a docker image
*/
def publishDockerImage(image_name, DOCKER_TAG) {
    echo "Logging to aws ecr"
    sh "aws ecr get-login --no-include-email --region us-east-1 | sh"
    echo "Tagging and pushing the safeops-api Docker Image"
    sh "docker tag ${image_name} ${DOCKER_REG}/${image_name}:${DOCKER_TAG}"
    sh "docker push ${DOCKER_REG}/${image_name}"
}
/*
    This is the main pipeline section with the stages of the CI/CD
 */
pipeline {
    options {
        // Build auto timeout
        timeout(time: 120, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(daysToKeepStr: '15', artifactDaysToKeepStr: '15'))
    }
    environment {
        IMAGE_NAME = 'demo-app'
        DOCKER_REG = "350473869200.dkr.ecr.us-east-1.amazonaws.com"
        PATH = "/Users/oiga/Library/Python/3.8/bin:$PATH"
    }    
    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'test', description: 'Git branch to build')
    }
    //all is built and run from the master
    agent { node { label 'master' } }
    // Pipeline stages
    stages {
        stage('Git clone and setup') {
            when {
                anyOf {
                    environment name: 'GIT_BRANCH', value: 'master'
                }
            }
            steps {
                script {
                    DOCKER_TAG = "prod"
                }
                echo "Check out code"
                checkout scm
                // Validate kubectl
                sh "kubectl cluster-info"
                // Helm version
                sh "helm version"
            }
        }
        stage('Build Docker Images') {
            when {
                anyOf {
                    environment name: 'GIT_BRANCH', value: 'master'
                }
            }
            parallel {
                stage('Build demo-app image') {
                    steps {
                        buildDockerImage("${IMAGE_NAME}", "Dockerfile", "${WORKSPACE}")
                    }
                }
            }
        }
        stage('Publish Docker Images') {
            when {
                anyOf {
                    environment name: 'GIT_BRANCH', value: 'master'
                }
            }
            parallel {
                stage('Publish demo-app image') {
                    steps {
                        publishDockerImage("${IMAGE_NAME}", "${DOCKER_TAG}")
                    }
                }
            }
        }
                stage('SafeOps') {
            steps{
                sh 'python3 -m pip install safeops-cli --user -U'
                sh 'safeops start-scans -a ndrar7Sa.rT0Sv2UKHhJ36sW6117p2t1vStHoZdpi'
                sh 'safeops get-results -a ndrar7Sa.rT0Sv2UKHhJ36sW6117p2t1vStHoZdpi'
            }
        }
        stage('Deploy') {
            parallel {
                stage('Deploying demo-app') {
                    steps {
                        script {
                            echo "Deploying k8s artifacts"
                            sh "kubectl apply -f k8s"
                        }
                    }
                }
            }
        }
    }
}
