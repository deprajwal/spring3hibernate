pipeline {
    agent { label 'assignment-4-agent' }

    parameters {
        booleanParam(
            name: 'SKIP_STABILITY',
            defaultValue: false,
            description: 'Skip Code Stability scan'
        )

        booleanParam(
            name: 'SKIP_QUALITY',
            defaultValue: false,
            description: 'Skip Code Quality analysis'
        )

        booleanParam(
            name: 'SKIP_COVERAGE',
            defaultValue: false,
            description: 'Skip Code Coverage analysis'
        )
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-11-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Code Checkout') {
            steps {
                checkout scm
                stash name: 'source-code', includes: '**/*', useDefaultExcludes: false
            }
        }

        stage('Parallel Code Scans') {
            parallel {

                stage('Code Stability') {
                    when {
                        expression {
                            !params.SKIP_STABILITY
                        }
                    }

                    steps {
                        dir('stability') {
                            deleteDir()
                            unstash 'source-code'

                            echo 'Running Code Stability scan...'

                            sh '''
                                mvn checkstyle:checkstyle
                            '''
                        }
                    }
                }

                stage('Code Quality') {
                    when {
                        expression {
                            !params.SKIP_QUALITY
                        }
                    }

                    steps {
                        dir('quality') {
                            deleteDir()
                            unstash 'source-code'

                            echo 'Running SonarQube Code Quality analysis...'

                            withEnv([
                                'JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64',
                                'PATH=/usr/lib/jvm/java-21-openjdk-amd64/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
                            ]) {
                                withSonarQubeEnv('SonarQube') {
                                    sh '''
                                        mvn -DskipTests \
                                           -Dsonar.projectKey=spring3hibernate \
                                           -Dsonar.projectName=spring3hibernate \
                                           sonar:sonar
                                    '''
                                }
                            }
                        }
                    }
                }

                stage('Code Coverage') {
                    when {
                        expression {
                            !params.SKIP_COVERAGE
                        }
                    }

                    steps {
                        dir('coverage') {
                            deleteDir()
                            unstash 'source-code'

                            echo 'Running JaCoCo Code Coverage analysis...'

                            sh '''
                                mvn clean \
                                   org.jacoco:jacoco-maven-plugin:0.8.13:prepare-agent \
                                   test \
                                   org.jacoco:jacoco-maven-plugin:0.8.13:report
                            '''
                        }
                    }
                }
            }
        }

        stage('Generate Reports') {
            steps {
                echo 'Publishing reports...'

                junit(
                    testResults: 'coverage/target/surefire-reports/*.xml',
                    allowEmptyResults: true
                )

                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'stability/target/site',
                    reportFiles: 'checkstyle.html',
                    reportName: 'Checkstyle Report'
                ])

                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'coverage/target/site/jacoco',
                    reportFiles: 'index.html',
                    reportName: 'JaCoCo Coverage Report'
                ])
            }
        }

        stage('Approval for Publication') {
            steps {
                input(
                    message: 'Approve artifact publication?',
                    ok: 'Approve'
                )
            }
        }

        stage('Publish Artifacts') {
            steps {
                dir('coverage') {
                    sh 'mvn package -DskipTests'

                    archiveArtifacts(
                        artifacts: 'target/*.war',
                        fingerprint: true
                    )
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        aborted {
            echo 'Pipeline aborted / publication denied.'
        }
    }
}
