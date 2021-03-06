pipeline {
    agent any

    stages{
        stage('Clone') {
            steps {
                script {
                    currentBuild.displayName = "#${env.BUILD_NUMBER} [ ${env.PROJECT_NAME} ]"

                    checkout scm
                    checkout poll:true, scm: [$class: 'GitSCM',
                    branches: [[name: 'refs/heads/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CleanCheckout'],
                    [$class: 'WipeWorkspace'],
                    [$class: 'CloneOption', depth: 0,
                    noTags: false, reference: '',
                    shallow: true],
                    [$class: 'RelativeTargetDirectory',
                    relativeTargetDir: 'app/'],
                    [$class: 'CleanBeforeCheckout']], submoduleCfg: [],
                    userRemoteConfigs: [[url: env.REPOSITORY]]]
                }
            }
        }

        stage('Remove Jenkinsfile') {
            steps {
                // remove jenkins file
                sh 'rm -rf app/Jenkinsfile app/.git';
            }
        }

        stage('Tests') {
            steps {
                echo "Analisando estrutura de testes..."
                script {
                    def status = sh(returnStatus: true, script: "if ! grep -q \"<!--teste_ok-->\" app/index.html; then echo \"ARQUIVO NÃO TEM ESTRUTURA DE TESTE VÁLIDO!\"; exit 1; fi")
                    if (status != 0) {
                        currentBuild.result = 'FAILED'
                        error "Failed, ARQUIVO NÃO TEM ESTRUTURA DE TESTE VÁLIDO..."
                    }
                }

            }
        }

        stage('Create project') {
            steps {
                // create project path
                sh "sudo mkdir -p /var/www/alunos/${env.PROJECT_NAME} && sudo chown jenkins:www-data /var/www/alunos/${env.PROJECT_NAME} -R";
                // create alias project
                sh "sudo echo \"Alias /${env.PROJECT_NAME} /var/www/alunos/${env.PROJECT_NAME}\" >> /var/www/sites.conf"
            }
        }

        stage('Clear project') {
            steps {
                // clear project
                sh "sudo rm -rf /var/www/alunos/${env.PROJECT_NAME}/*";
            }
        }

        stage('Publish') {
            steps {
                script {
                    // deploy project
                    sh "sudo cp -r ./app/* /var/www/alunos/${env.PROJECT_NAME}/";
                    // permission
                    sh "sudo chown jenkins:www-data /var/www/alunos -R";
                    // reload webserver
                    sh 'sudo /etc/init.d/apache2 reload'
                    // message
                    echo "===================================================================================="
                    echo "================>>>>>>>   ${env.PROJECT_NAME}"
                    echo "================>>>>>>>   Successfully published !!!"
                    echo "===================================================================================="
                }
            }
        }
    }
}