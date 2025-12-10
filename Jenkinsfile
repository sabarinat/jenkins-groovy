import groovy.json.JsonSlurper

def branchExists(String files, String branchName) {
    def status = sh(
        script: """
            cd "${files}"
            git rev-parse --verify "refs/heads/${branchName}" >/dev/null 2>&1
        """,
        returnStatus: true
    )
    return status == 0
}

def imageExists(String files, String tag, String image) {
    def status = sh(
        script: """
            cd "${files}"
            docker images "${image}:${tag}" | grep "${image}" >/dev/null 2>&1
        """,
        returnStatus: true
    )
    return status == 0
}

pipeline {
    agent any
    parameters {
        string(name: 'country', defaultValue: 'ind', description: 'Enter country')
        string(name: 'tag', description: 'Enter Tag')
        choice(name: 'env', description: 'Enter environments', choices: ['d1', 's1'])
    }
    environment {
        PORT = "3000"
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        TAG = "${params.tag}"
    }
    stages {
        stage('Clean workspace') {
            steps {
                sh "rm -rf /var/lib/jenkins/workspace/start/*"
            }
        }
        stage('Get all credentials from JSON') {
            steps {
                withCredentials([file(credentialsId: 'secret_v2', variable: 'KEY_FILE')]) {
                    script {
                        def fileContents = readFile(file: env.KEY_FILE)
                        def json = new JsonSlurper().parseText(fileContents)
                        env.docker_image = json.docker_image
                        json.each { key, value ->
                            env[key.toUpperCase()] = value
                        }
                    }
                }
            }
        }

        stage('Git Clone or Pull for kube') {
            steps {
                sh '''
                    if [ ! -d "kube-bas-learning" ]; then
                        git clone https://github.com/sabarinat/kube-bas-learning.git
                    else
                        cd kube-bas-learning
                        git checkout main
                        git pull
                    fi
                '''
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    sh "echo '${env.DOCKER_PASS}' | docker login -u '${env.DOCKER_USER}' --password-stdin"
                }
            }
        }

        stage('Git Clone or Pull for node with Docker') {
            steps {
                sh '''
                    echo $TAG
                    if [ ! -d "simple-node-docker" ]; then
                        git clone https://github.com/sabarinat/simple-node-docker.git
                    else
                        cd simple-node-docker
                        git checkout main
                        git pull
                    fi
                '''
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def branchName = "release-${TAG}"
                    def folder = "simple-node-docker"
                    if (imageExists(folder, TAG, env.DOCKER_IMAGE)) {
                        echo "Docker image '${env.DOCKER_IMAGE}:${TAG}' already exists"
                    } else {
                        sh """
                            cd ${folder}
                            git checkout tags/${TAG} -b release-${TAG}
                            docker build -t myapp:latest .
                            docker tag myapp:latest ${env.DOCKER_IMAGE}:${TAG}
                            docker push ${env.DOCKER_IMAGE}:${TAG}
                            git checkout main
                            git branch -d release-${TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def files = ['namespace.yaml','deployment.yaml','node-service.yaml', 'ingres.yaml']

                    files.each { file ->
                        def output = "generated-${file}"
                        // Replace placeholders using sed
                        sh """
                            cp kube-bas-learning/${file} ${output}
                            sed -i 's/{name_space}/${params.country}/g' ${output}
                            sed -i 's/{country}/${params.env}/g' ${output}
                            sed -i 's/{tag}/${params.tag}/g' ${output}
                        """
                        sh "kubectl apply -f ${output}"
                    }
                }
            }
        }
    }
}
