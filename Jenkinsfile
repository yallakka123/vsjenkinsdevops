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
                dir('ws') {
                    checkout([$class: 'GitSCM',
                        branches: [[name: '*/main']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [
                            [$class: 'SparseCheckoutPaths',
                             sparseCheckoutPaths: [
                                 [$class: 'SparseCheckoutPath', path: 'force-app/main/default/layouts']
                             ]
                            ]
                        ],
                        userRemoteConfigs: [[url: 'https://github.com/yallakka123/vsjenkinsdevops']]
                    ])
                }
            }
        }

        stage('Authenticate to Salesforce Orgs') {
            steps {
                sh '''
                    echo "Authenticating to Dev Org..."
                    sfdx force:auth:password:login \
                        --clientid $SF_CLIENT_ID_DEV \
                        --clientsecret $SF_CLIENT_SECRET_DEV \
                        --username $SF_DEV_USER_USR \
                        --password $SF_DEV_USER_PWD \
                        --instanceurl https://test.salesforce.com \
                        --setalias DevOrg

                    echo "Authenticating to Int Org..."
                    sfdx force:auth:password:login \
                        --clientid $SF_CLIENT_ID_INT \
                        --clientsecret $SF_CLIENT_SECRET_INT \
                        --username $SF_INT_USER_USR \
                        --password $SF_INT_USER_PWD \
                        --instanceurl https://test.salesforce.com \
                        --setalias IntOrg
                '''
            }
        }

        stage('Run Tests & Code Coverage on DEV') {
            steps {
                sh '''
                    echo "Running Apex Tests on DEV..."
                    sfdx force:apex:test:run \
                        --resultformat human \
                        --codecoverage \
                        --testlevel RunLocalTests \
