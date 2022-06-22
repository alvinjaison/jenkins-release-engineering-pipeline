def devRepo1 = [ 'bash', '-c', "aws ecr list-images --repository-name dev-repo1 --profile dev-aws-profile-name --query imageIds[*].imageTag --output text | sed 's/^/null\\n/g' | sed 's/\t/\\n/g'"]
def devRepo1Tags = devRepo1.execute().text
def devRepo2 = [ 'bash', '-c', "aws ecr list-images --repository-name dev-repo2 --profile dev-aws-profile-name --query imageIds[*].imageTag --output text | sed 's/^/null\\n/g' | sed 's/\t/\\n/g'"]
def devRepo2Tags = devRepo2.execute().text

def qaRepo1 = [ 'bash', '-c', "aws ecr list-images --repository-name qa-repo1 --profile qa-aws-profile-name --query imageIds[*].imageTag --output text | sed 's/^/null\\n/g' | sed 's/\t/\\n/g'"]
def qaRepo1Tags = qaRepo1.execute().text
def qaRepo2 = [ 'bash', '-c', "aws ecr list-images --repository-name qa-repo2 --profile qa-aws-profile-name --query imageIds[*].imageTag --output text | sed 's/^/null\\n/g' | sed 's/\t/\\n/g'"]
def qaRepo2Tags = qaRepo2.execute().text

def prodRepo1 = [ 'bash', '-c', "aws ecr list-images --repository-name production-company-information-service --profile prod-aws-profile-name --query imageIds[*].imageTag --output text | sed 's/^/null\\n/g' | sed 's/\t/\\n/g'"]
def prodRepo1Tags = prodRepo1.execute().text
def prodRepo2 = [ 'bash', '-c', "aws ecr list-images --repository-name production-configuration-service --profile prod-aws-profile-name --query imageIds[*].imageTag --output text | sed 's/^/null\\n/g' | sed 's/\t/\\n/g'"]
def prodRepo2Tags = prodRepo2.execute().text

pipeline {

    agent any

    stages {
        stage('Setup parameters') {
            steps {
                script {
                    properties([
                        parameters([
                            choice(
                                choices: ['Dev', 'QA', 'Prod'],
                                name: 'Environment'
                            )
                        ])
                    ])
                }
            }
        }

        stage("Interactive_Input") {
            steps {
                script {

                    // Variables for input
                    def inputConfig
                    def inputTest
                    def repo1ImageTags
                    def repo2ImageTags

                    if (params.Environment == 'Dev') {
                        echo 'Selected environment is Dev'
                        repo1ImageTags = devRepo1Tags
                        repo2ImageTags = devRepo2Tags
                    }  else if (params.Environment == 'QA') {
                        echo 'Selected environment is QA'
                        repo1ImageTags = qaRepo1Tags
                        repo2ImageTags = qaRepo2Tags
                    }  else  {
                        sh "echo 'Selected env is Production}'"
                        repo1ImageTags = prodRepo1Tags
                        repo2ImageTags = prodRepo2Tags
                    }

                    // Get the input
                    def userInput = input(
                            id: 'userInput', message: 'Select the versions you want to deploy:',
                            parameters: [
                                    choice(name: 'Repo1',
                                            choices: "${repo1ImageTags}",
                                            description: 'What is the release scope?'),
                                    choice(name: "repo2",
                                            choices: "${repo2ImageTags}",
                                            description: 'What is the release scope?'),
                            ])

                    // Save to variables. Default to empty string if not found.
                    inputrepo1 = userInput.repo1?:''
                    inputrepo2 = userInput.repo2?:''

                    // Echo to console
                    echo("repo1 tag selected: ${inputrepo1}")
                    echo("repo2 tag selected: ${inputrepo2}")
                    echo("Environment is: ${params.Environment}")

                    // Write to file
                    writeFile file: "inputData.txt", text: "Config=${inputConfig}\r\nTest=${inputTest}"

                    // Archive the file (or whatever you want to do with it)
                    archiveArtifacts 'inputData.txt'
                }
            }
        }

        stage('CheckIfDev') {
            when {
                expression { params.Environment == 'Dev' }
            }

            steps {
                script {
                    echo 'Deploying to Dev'
                    if (inputrepo1 == 'null') {
                        echo 'Seleted tag is null.. Skipping deployment for repo1'
                    }  else {
                        sh "echo 'selected tag is ${inputrepo1}'"
                        sh "export AWS_PROFILE=dev-profile"
                        sh "export AWS_PROFILE=dev-profile;aws eks update-kubeconfig --name devName --region region_name"
                        echo 'Deploying to Dev'
                //        sh "kubectl create namespace dev-test"
                        echo "=============================="
                        echo " Env : $params.Environment"
                        echo " Service : repo1"
                        echo " ImageTag : ${inputrepo1}"
                        echo "=============================="
                        sh ('cp -r kustomize/repo1 .')
                        sh ("sed -i 's/build/${inputrepo1}/' repo1/overlays/dev/kustomization.yaml")
                        sh ('kubectl apply -k repo1/overlays/dev -n dev-test')
                    }
                    echo 'Checking repo2 tag'
                    if (inputrepo2 == 'null') {
                        echo 'Seleted tag is null.. Skipping deployment for repo2'
                    }  else {
                        sh "echo 'selected tag is ${inputrepo2}'"
                        sh "export AWS_PROFILE=dev-aws-profile-name"
                        echo 'Deploying to Dev'
                        sh "export AWS_PROFILE=dev-aws-profile-name;aws eks update-kubeconfig --name dev-name --region ap-south-1"
                  //      sh "kubectl create namespace dev-test"
                        echo "=============================="
                        echo " Env : $params.Environment"
                        echo " Service : repo2"
                        echo " ImageTag : ${inputrepo2}"
                        echo "=============================="
                        sh ('cp -r kustomize/repo2 .')
                        sh ("sed -i 's/build/${inputrepo2}/' repo2/overlays/dev/kustomization.yaml")
                        sh ('kubectl apply -k repo2/overlays/dev -n dev-test')
                    }
                }
            }
        }

        stage('CheckIfQA') {
            when {
                expression { params.Environment == 'QA' }
            }
            steps {
                script {
                    echo 'Deploying to QA.. Checking repo1 tag'
                    if (inputrepo1 == 'null') {
                        echo 'Seleted tag is null.. Skipping deployment for repo1'
                    }  else {
                        sh "echo 'selected tag is ${inputrepo1}'"
                        echo 'Deploying to QA'
                        sh ('export AWS_PROFILE=qa-aws-profile-name')

                        sh "export AWS_PROFILE=qa-aws-profile-name;aws eks update-kubeconfig --name QA-name --region ap-south-1"

                       // sh "kubectl create namespace QA-test"
                        echo "=============================="
                        echo " Env : $params.Environment"
                        echo " Service : repo1"
                        echo " ImageTag : ${inputrepo1}"
                        echo "=============================="
                        sh ('cp -r kustomize/repo1/ .')
                        sh ("sed -i 's/build/${inputrepo1}/' repo1/base/deployment.yaml")
                        sh ('kubectl apply -k repo1/overlays/qa -n qa')
                    }
                    echo 'Checking repo2 tag'
                    if (inputrepo2 == 'null') {
                        echo 'Seleted tag is null.. Skipping deployment for repo2'
                    }  else {
                        sh "echo 'selected tag is ${inputrepo2}'"
                        echo 'Deploying to QA'
                        sh "export AWS_PROFILE=qa-aws-profile-name;aws eks update-kubeconfig --name QA-name --region ap-south-1"
                      //  sh "kubectl create namespace QA-test"
                        echo "=============================="
                        echo " Env : $params.Environment"
                        echo " Service : repo2"
                        echo " ImageTag : ${inputrepo2}"
                        echo "=============================="
                        sh ('cp -r kustomize/repo2/ .')
                        sh ("sed -i 's/build/${inputrepo2}/' repo2/base/deployment.yaml")
                        sh ('kubectl apply -k repo2/overlays/qa -n qa')
                    }
                }
            }
        }

        stage('CheckIfProduction') {
            when {
                expression {params.Environment == 'Production'}
            }

            steps {
                script {
                    echo 'Deploying to Production.. Checking repo1 tag'
                    if (inputrepo1 == 'null') {
                        echo 'Seleted tag is null.. Skipping deployment for repo1'
                    }  else {
                        sh "echo 'selected tag is ${inputrepo1}'"
                        sh "export AWS_PROFILE=prod-aws-profile-name"
                        sh "export AWS_PROFILE=prod-aws-profile-name;aws eks update-kubeconfig --name prod-name --region ap-south-1"
                        echo 'Deploying to Production'
                 //       sh "kubectl create namespace production-test"
                        echo "=============================="
                        echo " Env : $params.Environment"
                        echo " Service : repo1"
                        echo " ImageTag : ${inputrepo1}"
                        echo "=============================="
                        sh ('cp -r kustomize/repo1 .')
                        sh ("sed -i 's/build/${inputrepo1}/' repo1/overlays/prod/kustomization.yaml")
                        sh ('kubectl apply -k repo1/overlays/prod -n prod')
                    }
                    echo 'Checking repo2 tag'
                    if (inputrepo2 == 'null') {
                        echo 'Seleted tag is null.. Skipping deployment for repo2'
                    }  else {
                        sh "echo 'selected tag is ${inputrepo2}'"
                        sh "export AWS_PROFILE=prod-aws-profile-name"
                        sh "export AWS_PROFILE=prod-aws-profile-name;aws eks update-kubeconfig --name prod-name --region ap-south-1"
                        echo 'Deploying to Production'
                 //       sh "kubectl create namespace production-test"
                        echo "=============================="
                        echo " Env : $params.Environment"
                        echo " Service : repo2"
                        echo " ImageTag : ${inputrepo2}"
                        echo "=============================="
                        sh ('cp -r kustomize/repo2 .')
                        sh ("sed -i 's/build/${inputrepo2}/' repo2/overlays/prod/kustomization.yaml")
                        sh ('kubectl apply -k repo2/overlays/prod -n prod')
                    }
                }
            }
        }
    }
}
