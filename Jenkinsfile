pipeline {
    agent any
    triggers {
        pollSCM '*/1 * * * *'
    }
    environment {
        APPEXIST = 'no'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm: [ 
                    $class: 'GitSCM',
                    userRemoteConfigs: [[url: 'https://github.com/lutfi-ingram/laravelJenkins.git']],
                    branches: [[name: 'refs/heads/master']]
                ], poll: true
            }
        }
        stage('Create App') {
            steps {
                script {
                    openshift.withCluster() {
                        try {
                            def created = openshift.newApp( 'openshift/template.json', "-p", "NAME=${params.APPLICATION}"  )
                            timeout(1) {
                                created.logs('-f')
                            }
                        } catch (Exception ex) {
                            println(ex.getMessage())
                            println('writing amazing')
                            tmp_param = sh(script: 'yes', returnStdout: true).trim()
                            env.APPEXIST = tmp_param
                        }
                    }
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    openshift.withCluster() {
                        println("ada apa")
                        println(env.APPEXIST)
                        if (env.APPEXIST == 'yes'){
                            def bc = openshift.selector( "bc/${params.BC_NAME}" )
                            def result = bc.startBuild()
                            timeout(10) {
                                result.logs('-f')
                            }
                        }
                    }
                }
            }
        }
        stage('Rollout') {
            steps {
                script {
                    openshift.withCluster() {
                        def dc = openshift.selector( "dc/${params.APPLICATION}" )
                        try {
                            def rollout = dc.rollout()
                            def resultRollout = rollout.latest()
                    } catch (Exception ex) {
                            println(ex.getMessage())
                        }
                    }
                }
            }
        }
    }
}
