@Library('workflow-api-library') _
pipeline {
    //agent {label 'fmci70480'}
    agent any
    environment {
        TIME = Calendar.getInstance().getTime().format('YYYYMMdd_hhmmss',TimeZone.getTimeZone('CST'))
        TOP_FOLDER_NAME = 'HVM_xactor_' +"${TIME}"
        SPARK_SAMPLE_FOLDER =  'sample_project'
        HVM_FOLDER = 'hvm_xtor'
        PATH_SPARK_SAMPLE_FOLDER = "${env.WORKSPACE}" + "/" + "${TOP_FOLDER_NAME}" + "/" + "${SPARK_SAMPLE_FOLDER}"
        PATH_HVM_FOLDER = "${env.WORKSPACE}" + "/" + "${TOP_FOLDER_NAME}" + "/" + "${HVM_FOLDER}"
        TMP_FOLDER = 'HVM_xactor_' +"${TIME}" + "@tmp"
        TMP_FOLDER2 = 'HVM_xactor_' +"${TIME}" + "@tmp" + "@tmp"
    }
    stages {
        stage ('Checkout ') {
            steps {
                script {
                    postWorkflowRun()
                }
                echo "Checkout"
                dir("${TOP_FOLDER_NAME}") {
                    dir("${SPARK_SAMPLE_FOLDER}"){
                        sh 'pwd'
                        git credentialsId: 'sys_hvmxtor_git_token', url: 'https://github.com/intel-innersource/frameworks.validation.spark.sample-project'
                        sh '''
                            git checkout spark-1.11.10
                            echo '\n  hvm_xtor:\n    PATH: $RTL_CAD_ROOT/spark/hvm_xtor/release-1.25.0_abi_1/\n\n  tap_xtor:\n    PATH: $RTL_CAD_ROOT/spark/tap_xtor/spark-1.10.0_sles12_fix' >> cfg/repositories.yml
                            echo replacing files
                            rm -v cfg/buildProjectCfg.yml
                            cp -v /nfs/fm/disks/mpe_emu_002/jenkins/unitests/source_files/buildProjectCfg.yml cfg/
                        '''
                    }
                }
            }
        }
        stage('Build '){
            steps{
                echo 'Build'
                dir("${TOP_FOLDER_NAME}") {
                    dir("${SPARK_SAMPLE_FOLDER}"){
                        sh '''
                            ps -p $$
                            pwd
                            tcsh -c 'source ./source_common_env.csh; setenv MODEL_ROOT $PATH_SPARK_SAMPLE_FOLDER; echo $MODEL_ROOT; emubuild -cfg VCS -cfg- -no_parallel'
                        '''
                    }
                }
            }
        }
        
        stage('Tests '){
            steps{
                echo 'Run Tests'
                echo 'Building tests'
                dir("${TOP_FOLDER_NAME}") {
                    echo PATH_SPARK_SAMPLE_FOLDER
                    sh '''
                        echo Building tests
                        cd sample_project
                        export MODEL_ROOT=$PWD
                        echo $MODEL_ROOT
                        cp -r /nfs/fm/disks/mpe_emu_002/whidberc/hvm_xtor_tests/backup/hvm_xtor_test .
                        cd hvm_xtor_test
                        pwd
                        cmake .
                        cmake --build .
                        cd test_hvm_xtor/
                    '''
                }
                echo 'Running tests '
                dir("${TOP_FOLDER_NAME}"){
                    sh '''
                        cd sample_project/hvm_xtor_test/test_hvm_xtor/
                        pwd
                        ./run --gtest_output="xml:testresults.xml" --gtest_filter=-SimicsTest.*
                    '''
                    dir("sample_project"){
                        dir("hvm_xtor_test"){
                            dir('test_hvm_xtor'){
                                sh 'pwd'
                                junit '**/testresults.xml'
                            }
                        }
                    }
                    sh '''
                        cd sample_project/hvm_xtor_test/
                        lcov --capture --directory CMakeFiles/ --output-file coverage.info
                        lcov --remove coverage.info '/nfs/site/eda/*' '/nfs/site/itools/*' "/usr/intel/pkgs/*" --output-file coverage.info
                        genhtml coverage.info --output-directory out
                    '''
                    dir("sample_project"){
                        dir("hvm_xtor_test"){
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'out', reportFiles: 'index.html', reportName: 'Coverage Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
                
            }
        }
    }
    post {
        always {
            echo "${TMP_FOLDER}"
            dir("${TMP_FOLDER}") {
                sh "pwd"
                sh "rm -rf ${TMP_FOLDER2}"
                deleteDir()
            }
        }
    }
}
