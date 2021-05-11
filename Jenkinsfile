pipeline {
    agent any

    stages{
        stage("Warm Up") {
            steps {
                script {
                    currentBuild.displayName = "#${env.BUILD_NUMBER} [ ${env.PROJECT_NAME} ]"
                }
            }
        }

        stage('Clone') {
            steps {
                script {
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

        stage('Create project') {
            steps {
                // create project path
                sh "sudo mkdir -p /var/www/alunos/${env.PROJECT_NAME} && sudo chown jenkins:www-data /var/www/alunos/${env.PROJECT_NAME} -R";
                // create webserver path
                sh "sudo cp 000-default.conf /etc/apache2/sites-available/000-default.conf && sudo chown root:root /etc/apache2/sites-available/000-default.conf";
            }
        }

        stage('Update list') {
            steps {
                // update list sites
    sh """
#!/bin/bash
sudo echo "" > /var/www/sites.conf
LIST_SITES=""
for entry in "/var/www/alunos"/*
do
  PROJECT=\$(basename \$entry)
  NEW_LINE="Alias /\$PROJECT \"\$entry\""
  LIST_SITES=\"\${LIST_SITES}<li><a href=\"/\$PROJECT\" target=\"_parent\">\$PROJECT</a></li>\"

  sudo echo "\$NEW_LINE" >> /var/www/sites.conf
done

sudo echo "" > /var/www/html/list.html
sudo echo "\$LIST_SITES" > /var/www/html/list.html
    """
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