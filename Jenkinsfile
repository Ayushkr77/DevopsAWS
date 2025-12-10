pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET  = 'ayush-emp-lambda-artifacts'
        LAMBDA_FN  = 'EmpLambda'
        ZIP_NAME   = 'lambda.zip'
        S3_KEY     = "emp-artifacts/${BUILD_NUMBER}/${ZIP_NAME}"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'dev', url: 'https://github.com/Ayushkr77/DevopsAWS.git'
            }
        }

        stage('Package Lambda Code') {
            steps {
                sh '''
                    cd lambda
                    rm -f ${ZIP_NAME}
                    zip ${ZIP_NAME} lambda_function.py
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'awscreds',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        export AWS_DEFAULT_REGION=${AWS_REGION}
                        aws s3 cp lambda/${ZIP_NAME} s3://${S3_BUCKET}/${S3_KEY}
                    '''
                }
            }
        }

        stage('Deploy to Lambda') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'awscreds',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        export AWS_DEFAULT_REGION=${AWS_REGION}
                        aws lambda update-function-code \
                            --function-name ${LAMBDA_FN} \
                            --s3-bucket ${S3_BUCKET} \
                            --s3-key ${S3_KEY}
                    '''
                }
            }
        }
    }
}
