void Deploy(env_type) {
    withEnv (
        [
            "DOCKER_HOST=tcp://$env_type"
        ]
    ) {
        withCredentials (
            [
                usernamePassword(
                    credentialsId: 'registry_creds',
                    usernameVariable: 'DR_USER',
                    passwordVariable: 'DR_PASS'
                )
            ]
        ) {
            sh 'docker login -u ${DR_USER} -p ${DR_PASS}'
            sh 'docker service update --image $registry/frontend:$BUILD_NUMBER frontend'
            //sh 'docker service create --with-registry-auth --name frontend -p 80:80 $registry/frontend:$BUILD_NUMBER'
        }
    }
}

pipeline {
    environment {
        registry = "sergeykudelin"
        registry_url = "https://hub.docker.com"
        registry_creds = "registry_creds"
        dev_env = "dev.sergeykudelin.pp.ua:2375"
        stage_env = "stage.sergeykudelin.pp.ua:2375"
        prod_env = "prod.sergeykudelin.pp.ua:2375"
    }
    agent any
    stages {
        stage("Build") {
            steps {
                echo "Here we are building"
                script {
                    echo "Building sergeykudelin/frontend:${BUILD_NUMBER}"
                    AppImage = docker.build registry + '/frontend:latest'
                    docker.withRegistry('https://registry.hub.docker.com', 'registry_creds') {
                        AppImage.push("${env.BUILD_NUMBER}")            
                        AppImage.push("latest")
                    }
                }
            }
        }
        stage("Deploy regular branch") {
            when {
                allOf {
                    not {branch 'master'}
                    not {branch 'develop'}
                    not {buildingTag()}
                }
            }
            steps {
                echo "Here we are, deploying regular branch $BRANCH_NAME"
                Deploy(dev_env)
            }
        }
        stage("Deploy develop branch") {
            when {
                branch 'develop'
            }
            steps {
                echo "Here we are, deploying develop branch $BRANCH_NAME"
                Deploy(stage_env)
            }
        }
        stage("Deploy master branch") {
            when {
                branch 'master'
                buildingTag()
            }
            steps {
                input('Deploy master to prod?')
                echo "Here we are, deploying master branch $BRANCH_NAME"
                Deploy(prod_env)
            }
        }
    }
}