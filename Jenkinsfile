@Library('monolithic') _

pipeline {
    agent any
    environment {
        GIT_REPO_URL = 'https://github.com/jessiebrownleo/html-sample.git'
        GIT_BRANCH = 'main'
        INFRA_BRANCH = 'main'
        GITHUB_TOKEN = 'ghp_zswpeyptplu7htqpfl220ntnhrwtub39688i'
        WEBHOOK_URL = 'https://jenkins.cloudinator.cloud/github-webhook/'
        DOCKER_IMAGE_NAME = 'sovanra/testhtml'
        DOCKER_IMAGE_TAG = '${BUILD_NUMBER}'
        DOCKER_CREDENTIALS_ID = 'docker'
        GIT_INFRA_URL = 'https://github.com/sovanra-ruos/infra.git'
        INVENTORY_FILE = 'inventory/inventory.ini'
        PLAYBOOK_FILE = 'playbooks/deploy-with-k8s.yml'
        HELM_FILE = 'playbooks/setup-helm.yml'
        APP_NAME = 'testhtml'
        FILE_Path = 'deployments/${APP_NAME}'
        IMAGE = "${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
        NAMESPACE = 'testhtml'
        DOMAIN_NAME = 'tsesthtml.cloudinator.cloud'
        EMAIL = 'vannraruos@gmail.com'
        TRIVY_SEVERITY = 'HIGH,CRITICAL'
        TRIVY_EXIT_CODE = '0'
        TRIVY_IGNORE_UNFIXED = 'true'
        VULN_THRESHOLD = '500'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: env.GIT_BRANCH, url: env.GIT_REPO_URL
            }
        }
        stage('Generate Dockerfile') {
            steps {
                script {
                    projectInfo = detectProjectType("${env.WORKSPACE}")
                }
            }
        }
        stage('Update Dependencies') {
            steps {
                script {
                    updateDependencies()
                }
            }
        }
        stage('Docker Login') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker', variable: 'DOCKER_PWD')]) {
                        sh 'echo $DOCKER_PWD | docker login -u sovanra --password-stdin'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerBuild("${DOCKER_IMAGE_NAME}", "${DOCKER_IMAGE_TAG}")
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                script {
                    def vulnerabilitiesCount = trivyScan(
                        DOCKER_IMAGE_NAME,
                        DOCKER_IMAGE_TAG,
                        TRIVY_SEVERITY,
                        TRIVY_EXIT_CODE,
                        TRIVY_IGNORE_UNFIXED,
                        VULN_THRESHOLD.toInteger()
                    )
                    echo 'Total vulnerabilities found: ${vulnerabilitiesCount}'
                }
            }
        }
        stage('Push Image to Registry') {
            steps {
                script {
                    dockerPush("${DOCKER_IMAGE_NAME}", "${DOCKER_IMAGE_TAG}")
                }
            }
        }
        stage('Clone infra') {
            steps {
                git branch: env.INFRA_BRANCH, url: env.GIT_INFRA_URL
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    deployToKubernetes(
                        INVENTORY_FILE,
                        PLAYBOOK_FILE,
                        APP_NAME,
                        IMAGE,
                        NAMESPACE,
                        FILE_Path,
                        DOMAIN_NAME,
                        EMAIL,
                        GIT_REPO_URL,
                        GIT_BRANCH
                    )
                }
            }
        }
        stage('set up helm') {
            steps {
                setUpHelm(
                    INVENTORY_FILE,
                    HELM_FILE,
                    APP_NAME,
                    DOCKER_IMAGE_NAME,
                    NAMESPACE,
                    DOCKER_IMAGE_TAG,
                    DOMAIN_NAME,
                    GIT_REPO_URL,
                        GIT_BRANCH
                )
            }
        }
        stage('Setup GitHub Webhook') {
            steps {
                script {
                    createGitHubWebhook(env.GIT_REPO_URL, env.WEBHOOK_URL, env.GITHUB_TOKEN)
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
