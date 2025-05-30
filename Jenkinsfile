pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'                  // Change as per your region
        LAMBDA_FUNCTION_NAME = 'my-demo-lambda'  // Your Lambda function name
        ZIP_FILE = 'lambda_function.zip'
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone your GitHub repo
                checkout scm
            }
        }

        stage('Package Lambda') {
            steps {
                // Zip your lambda_function.py (adjust if you have more files)
                sh """
                    zip -r ${ZIP_FILE} lambda_function.py
                """
            }
        }

        stage('Deploy Lambda') {
            steps {
                script {
                    // Deploy Lambda function using AWS CLI
                    // First try to update the function code, if it fails create the function

                    def updateStatus = sh(
                        script: """
                        aws lambda update-function-code --function-name ${LAMBDA_FUNCTION_NAME} --zip-file fileb://${ZIP_FILE} --region ${AWS_REGION}
                        """,
                        returnStatus: true
                    )

                    if (updateStatus != 0) {
                        echo "Function does not exist. Creating new Lambda function..."

                        // You must have an IAM role ARN ready for Lambda execution
                        def lambdaRoleArn = 'arn:aws:iam::123456789012:role/lambda-execution-role'  // replace this

                        sh """
                        aws lambda create-function --function-name ${LAMBDA_FUNCTION_NAME} \
                        --runtime python3.9 --role ${lambdaRoleArn} \
                        --handler lambda_function.lambda_handler --zip-file fileb://${ZIP_FILE} \
                        --region ${AWS_REGION}
                        """
                    } else {
                        echo "Lambda function updated successfully."
                    }
                }
            }
        }
    }
}

