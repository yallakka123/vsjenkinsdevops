import groovy.json.JsonSlurperClassic

pipeline {
    agent any

    environment {
        // GitHub
        GITHUB_PAT = credentials('github-pat')

        // Salesforce DEV Org
        SF_DEV_USER_USR = credentials('sf-dev-username')
        SF_DEV_USER_PWD = credentials('sf-dev-password')
        SF_CLIENT_ID_DEV = credentials('sf-client-id-dev')
        SF_CLIENT_SECRET_DEV = credentials('sf-client-secret-dev')

        // Salesforce INT Org
        SF_INT_USER_USR = credentials('sf-int-username')
        SF_INT_USER_PWD = credentials('sf-int-password')
        SF_CLIENT_ID_INT = credentials('sf-client-id-int')
        SF_CLIENT_SECRET_INT = credentials('sf-client-secret-int')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Node & SFDX') {
            steps {
                bat '''
                    echo Installing Salesforce CLI and Delta Plugin...
                    npm install --global sfdx-cli
                    npm install --global sfdx-git-delta
                    sfdx plugins:install sfdx-git-delta
                    sfdx --version
                '''
            }
        }

        stage('Authenticate to Salesforce Orgs') {
            steps {
                bat """
                    echo Authenticating to Dev Org...
                    sfdx force:auth:password:login ^
                        --clientid %SF_CLIENT_ID_DEV% ^
                        --clientsecret %SF_CLIENT_SECRET_DEV% ^
                        --username %SF_DEV_USER_USR% ^
                        --password %SF_DEV_USER_PWD% ^
                        --instanceurl https://test.salesforce.com ^
                        --setalias DevOrg

                    echo Authenticating to Int Org...
                    sfdx force:auth:password:login ^
                        --clientid %SF_CLIENT_ID_INT% ^
                        --clientsecret %SF_CLIENT_SECRET_INT% ^
                        --username %SF_INT_USER_USR% ^
                        --password %SF_INT_USER_PWD% ^
                        --instanceurl https://test.salesforce.com ^
                        --setalias IntOrg
                """
            }
        }

        stage('Run Tests & Code Coverage on DEV') {
            steps {
                bat """
                    echo Running Apex Tests on DEV...
                    sfdx force:apex:test:run ^
                        --resultformat human ^
                        --codecoverage ^
                        --testlevel RunLocalTests ^
                        --wait 10 ^
                        -u DevOrg
                """
            }
        }

        stage('Validate Deployment to INT (Delta Only)') {
            steps {
                bat """
                    echo Generating Delta Changes...
                    if not exist delta mkdir delta
                    sfdx sgd:source:delta --to HEAD --from HEAD~1 --output delta

                    echo Validating Delta Deployment on INT...
                    sfdx force:source:deploy ^
                        --sourcepath delta ^
                        --checkonly ^
                        --testlevel RunLocalTests ^
                        -u IntOrg
                """
            }
        }

        stage('Deploy Delta Changes to INT') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                bat """
                    echo Deploying Delta Changes to INT...
                    sfdx force:source:deploy ^
                        --sourcepath delta ^
                        --testlevel RunLocalTests ^
                        -u IntOrg
                """
            }
        }
    }
}
