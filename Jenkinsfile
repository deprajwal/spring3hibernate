pipeline {

    agent {
        label 'assignment-4-agent'
    }

    parameters {

        booleanParam(
            name: 'SKIP_STABILITY',
            defaultValue: false,
            description: 'Skip Code Stability Scan (Checkstyle)'
        )

        booleanParam(
            name: 'SKIP_QUALITY',
            defaultValue: false,
            description: 'Skip Code Quality Analysis (SonarQube)'
        )

        booleanParam(
            name: 'SKIP_COVERAGE',
            defaultValue: false,
            description: 'Skip Code Coverage Analysis (JaCoCo)'
        )
    }

    environment {

        JAVA_HOME = '/usr/lib/jvm/java-11-openjdk-amd64'

        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        // ============================================================
        // 1. CODE CHECKOUT
        // ============================================================

        stage('Code Checkout') {

            steps {

                echo 'Checking out source code...'

                checkout scm

                stash(
                    name: 'source-code',
                    includes: '**/*',
                    useDefaultExcludes: false
                )

                echo 'Source code checkout completed.'
            }
        }


        // ============================================================
        // 2. PARALLEL CODE SCANS
        // ============================================================

        stage('Parallel Code Scans') {

            parallel {

                // ----------------------------------------------------
                // CODE STABILITY - CHECKSTYLE
                // ----------------------------------------------------

                stage('Code Stability') {

                    when {

                        expression {
                            return !params.SKIP_STABILITY
                        }
                    }

                    steps {

                        dir('stability') {

                            deleteDir()

                            unstash 'source-code'

                            echo 'Running Code Stability Scan using Checkstyle...'

                            sh '''
                                echo "Java version:"
                                java -version

                                echo "Maven version:"
                                mvn -version

                                mvn checkstyle:checkstyle
                            '''

                            echo 'Code Stability Scan completed.'
                        }
                    }
                }


                // ----------------------------------------------------
                // CODE QUALITY - SONARQUBE
                // ----------------------------------------------------

                stage('Code Quality') {

                    when {

                        expression {
                            return !params.SKIP_QUALITY
                        }
                    }

                    steps {

                        dir('quality') {

                            deleteDir()

                            unstash 'source-code'

                            echo 'Running Code Quality Analysis using SonarQube...'

                            /*
                             * Java 11 is used here because the project
                             * contains an old FindBugs plugin which has
                             * compatibility issues with Java 21.
                             */

                            withEnv([
                                'JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64',
                                "PATH=/usr/lib/jvm/java-11-openjdk-amd64/bin:${env.PATH}"
                            ]) {

                                withSonarQubeEnv('SonarQube') {

                                    sh '''
                                        echo "Java version:"
                                        java -version

                                        echo "Maven version:"
                                        mvn -version

                                        mvn -DskipTests \
                                        compile \
                                        org.sonarsource.scanner.maven:sonar-maven-plugin:3.10.0.2594:sonar \
                                        -Dsonar.projectKey=spring3hibernate \
                                        -Dsonar.projectName=spring3hibernate
                                    '''
                                }
                            }

                            echo 'Code Quality Analysis completed.'
                        }
                    }
                }


                // ----------------------------------------------------
                // CODE COVERAGE - JACOCO
                // ----------------------------------------------------

                stage('Code Coverage') {

                    when {

                        expression {
                            return !params.SKIP_COVERAGE
                        }
                    }

                    steps {

                        dir('coverage') {

                            deleteDir()

                            unstash 'source-code'

                            echo 'Running Code Coverage Analysis using JaCoCo...'

                            sh '''
                                echo "Java version:"
                                java -version

                                echo "Maven version:"
                                mvn -version

                                mvn clean \
                                org.jacoco:jacoco-maven-plugin:0.8.13:prepare-agent \
                                test \
                                org.jacoco:jacoco-maven-plugin:0.8.13:report
                            '''

                            echo 'Code Coverage Analysis completed.'
                        }
                    }
                }
            }
        }


        // ============================================================
        // 3. GENERATE REPORTS
        // ============================================================

        stage('Generate Reports') {

            steps {

                echo 'Generating reports...'


                // ----------------------------------------------------
                // JUNIT TEST REPORT
                // ----------------------------------------------------

                junit(
                    testResults: 'coverage/target/surefire-reports/*.xml',
                    allowEmptyResults: true
                )


                // ----------------------------------------------------
                // CHECKSTYLE REPORT
                // ----------------------------------------------------

                publishHTML(
                    target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'stability/target/site',
                        reportFiles: 'checkstyle.html',
                        reportName: 'Checkstyle Report'
                    ]
                )


                // ----------------------------------------------------
                // JACOCO REPORT
                // ----------------------------------------------------

                publishHTML(
                    target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'coverage/target/site/jacoco',
                        reportFiles: 'index.html',
                        reportName: 'JaCoCo Coverage Report'
                    ]
                )

                echo 'Reports generated successfully.'
            }
        }


        // ============================================================
        // 4. APPROVAL BEFORE PUBLICATION
        // ============================================================

        stage('Approval for Publication') {

            steps {

                echo 'Waiting for approval before publishing artifact...'

                input(
                    message: 'Approve artifact publication?',
                    ok: 'Approve'
                )

                echo 'Artifact publication approved.'
            }
        }


        // ============================================================
        // 5. PUBLISH ARTIFACT
        // ============================================================

        stage('Publish Artifacts') {

            steps {

                dir('coverage') {

                    echo 'Building WAR artifact...'

                    sh '''
                        mvn package -DskipTests
                    '''

                    echo 'Publishing WAR artifact...'

                    archiveArtifacts(
                        artifacts: 'target/*.war',
                        fingerprint: true
                    )

                    echo 'Artifact published successfully.'
                }
            }
        }
    }


    // ================================================================
    // POST BUILD ACTIONS
    // ================================================================

    post {

        success {

            echo '========================================'
            echo 'PIPELINE SUCCESS'
            echo 'Artifact published successfully.'
            echo '========================================'

            /*
             * Slack notification will be configured later.
             */

            /*
             * Email notification will be configured later.
             */
        }


        failure {

            echo '========================================'
            echo 'PIPELINE FAILED'
            echo 'Please check Jenkins console output.'
            echo '========================================'

            /*
             * Slack notification will be configured later.
             */

            /*
             * Email notification will be configured later.
             */
        }


        aborted {

            echo '========================================'
            echo 'PIPELINE ABORTED'
            echo '========================================'
        }
    }
}
