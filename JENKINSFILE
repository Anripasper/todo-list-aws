pipeline {
    agent any
    stages {

        stage('Get Code') {
            steps {
               
               git branch: 'master',
               url: 'https://github.com/Anripasper/todo-list-aws.git',
               credentialsId: 'GitTOKEN'

                }
            }
    stage('Set Environment') {
    steps {
        sh '''
             python3 -m venv .venv
            echo "==== CREAR VENV ===="
            python3 -m venv venv

            . venv/bin/activate
            pip install --upgrade pip
            pip install flake8 bandit 
            pip install pytest requests
        '''
    }
}

        stage('Deploy') {
            
            steps {
                sh '''
                    aws sts get-caller-identity
                    sam build
                    sam validate --region us-east-1
                   sam deploy --config-env production \
                --stack-name production-todo-list-aws \
                --region us-east-1 \
                --resolve-s3 \
                --capabilities CAPABILITY_IAM \
                --no-disable-rollback \
                --parameter-overrides Stage="production" \
                --save-params \
                --no-confirm-changeset \
                --no-fail-on-empty-changeset \
                '''
            }
        }
          
 
        
        stage('Rest Test') {
            steps {
                sh'''
               
                   . venv/bin/activate
                   export BASE_URL=$(aws cloudformation describe-stacks --stack-name production-todo-list-aws --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text)
                   echo $BASE_URL
                  
                   echo "-------ejecutar pruebas----------"
                    pytest --junitxml=result-rest.xml test/integration/todoApiTest.py -k 'test_api_listtodos or test_api_gettodo'
                  '''
                   junit 'result-rest.xml'
             
            }
        }
        
    }}
