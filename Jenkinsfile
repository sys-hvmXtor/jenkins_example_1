@Library('workflow-api-library') _
pipeline {
    agent {label 'fmci70480'}
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
        stage ('Checkout') {
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
        stage('Build'){
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
    }
}
