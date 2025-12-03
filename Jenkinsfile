import groovy.json.JsonSlurper

def branchExists(String files, String branchName) {
    // returnStatus: 0 if branch exists, 1 if not
    def status = bat(
        script: "cd ${files} && git rev-parse --verify ${branchName}",
        returnStatus: true
    )
    
    return status == 0
}

def imageExists(String files, String tag, String image) {
    // returnStatus: 0 if branch exists, 1 if not
    def status = bat(
        script: "cd ${files} && docker images ${image}:${tag}",
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
            KUBECONFIG = 'C:\\Users\\Sabar\\.kube\\config'
            TAG = "${params.tag}"
            
       }
    stages {
        
        stage('get all cred from json') {
            steps {
                    withCredentials([file(credentialsId: 'secret_v2', variable: 'KEY_FILE')]) {
                     script {
                         def fileContents = readFile(file: env.KEY_FILE)
                         def json = new JsonSlurper().parseText(fileContents)
                        //  bat "echo %json.docker_image%"
                        // env = json
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
                    // Clone if folder doesn't exist, otherwise pull
                    bat '''
                    if not exist "kube-bas-learning" (
                        git clone https://github.com/sabarinat/kube-bas-learning.git
                    ) else (
                        cd kube-bas-learning
                        git checkout main
                        git pull
                    )
                    
                    '''
                }
            }
        
        stage('Docker Login') {
            steps {
                script {
                    bat "echo ${env.DOCKER_PASS} | docker login -u ${env.DOCKER_USER} --password ${env.DOCKER_PASS}"
                }
            }
        }
        
        stage('Git Clone or Pull for node with docker') {
                steps {
                    // Clone if folder doesn't exist, otherwise pull
                    bat "echo --- $tag"
                    bat '''
                    echo %TAG%
                    if not exist "simple-node-docker" (
                       git clone https://github.com/sabarinat/simple-node-docker.git
                    ) else (
                        cd simple-node-docker
                        git checkout main
                        git pull
                    )
                    '''
                }
            }
            
        stage('Docker push image') {
                steps {
                    // Clone if folder doesn't exist, otherwise pull
                    script {
                    def branchName = "release-%TAG%"
                    bat "cd simple-node-docker"
                    def folder = "simple-node-docker"
                    if (imageExists(folder, TAG, env.DOCKER_IMAGE)) {
                        echo "Branch '${branchName}' exists in local repo"
                    } else {
                        bat '''cd simple-node-docker
                            git checkout tags/%TAG% -b release-%TAG%"
                            docker build -t myapp:latest .
                           docker tag myapp:latest %DOCKER_IMAGE%:%TAG%
                           docker push %DOCKER_IMAGE%:%TAG%
                           git checkout main
                           git branch -d release-%TAG%"
                            '''
                    }
                }
                }
            }
        
        stage('Kubernetes') {
            steps { 
                script {
                    def files = ['namespace.yaml','deployment.yaml','node-service.yaml', 'ingres.yaml']
    
                        files.each { file ->
                        echo "Processing file: ${file}"

                        // Create temp file to avoid corrupting original YAML
                        def output = "generated-${file}"
                        echo "Processing file: ---- ${file}"
                        // Replace placeholder
                        powershell """
                            (Get-Content kube-bas-learning/${file}) -replace '{name_space}', '${params.country}' |
                            Set-Content ${output}
                            (Get-Content ${output}) -replace '{country}', '${params.env}' |
                            Set-Content ${output}
                            (Get-Content ${output}) -replace '{tag}', '${params.tag}' |
                            Set-Content ${output}
                        """
                         echo "Processing file: ---- ${file}"
                        // Apply using kubectl
                        bat """
                            kubectl apply -f ${output}
                        """
                }
                }
            }
        }
    }
}