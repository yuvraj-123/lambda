def getDevVersion() {
    script {
        return sh(script: "git rev-parse --short=7 HEAD ", returnStdout: true)
    }
}

def getBranchName() {
    script {
        return sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true)
    }
}


def buildlambda(dir) {
    if ("${BUILD_VERSION}" == "") {
        BUILD_VERSION = getDevVersion().trim()
    }
    sh "cd ${dir}/ && sam build "
}

def deploylambda(dir) {
    if ("${BUILD_VERSION}" == "") {
        BUILD_VERSION = getDevVersion().trim()
    }
    sh "cd ${dir}/ && sam deploy --no-confirm-changeset --no-fail-on-empty-changeset --template-file template.yaml --config-file ${ENVIRONMENT}-config.toml --config-env ${ENVIRONMENT} --s3-bucket ${s3_bucket}" 
    }


pipeline {
    parameters {
        text(name: 'lambdas', defaultValue:'lambda1\nlambda2', description: 'lambdas to build')
        string(name: 'BUILD_VERSION', defaultValue:'', description: 'build version')
        string(name: 's3_bucket', defaultValue: 'cloudfront030702', description: 's3 bukcet for lambda')
        string(name: 'branch_name', defaultValue: 'main', description: '')
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: '')
    }
    environment {
        BRANCH_NAME = "${params.branch_name}".trim()
        LAMBDAS = "${lambdas}"
    }
    agent any
    stages {
        // stage('Edit git config file'){
        //     steps{                
        //         sh 'git config --global url.ssh://git@github.com/.insteadOf https://github.com/'
        //     }
        // }

        stage('Build Lambda') {
            steps {
                script {
                    lambdas = "${params.lambdas}".split('\n')
                    def branches = [:]
                    for (lambda in lambdas) {
                        def lamb = lambda
                        branches["Build ${lambda}"] = {buildlambda(lamb)}
                    }
                    parallel branches
                }
            }
        }
        stage('Deploy Lambda') {
            steps {
                script {
                    lambdas = "${params.lambdas}".split('\n')
                    def branches = [:]
                    for (lambda in lambdas) {
                        def lamb = lambda
                        branches["Build ${lambda}"] = {deploylambda(lamb)}
                    }
                    parallel branches
                }
            }
        }
    }
}