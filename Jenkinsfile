pipeline {
    // 스테이지 별로 다른 거
    agent any // 어떤 젠킨스를 쓸건가
    // cron 얼마 주기로 파이프라인을 돌릴거냐
    triggers {
        pollSCM('*/3 * * * *')
    }
    // 파이프라인 안에서 쓸 환경변수
    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId') // jenkins에서 가져올 aws id
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey') // jenikns에서 가져올 aws key
      AWS_DEFAULT_REGION = 'ap-northeast-2' // 서울
      HOME = '.' // Avoid npm root owned
    }
    // 하나의 단계를 의미
    stages {
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            // repo 풀 땡김
            steps {
                echo 'Clonning Repository'
                // 내 깃 url
                git url: 'https://github.com/ck-kor/jenkins_test_repo.git',
                    branch: 'main',
                    credentialsId: 'gittest' // jenkins git 생성한id?
            }
            
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Successfully Cloned Repository'
                }

                always {
                  echo "i tried..."
                }
                // post가 다끝났다 
                cleanup {
                  echo "after all other post condition"
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){ // root > website에 가서 s3에 올리겠다. 
                sh '''
                aws s3 sync ./ s3://jenkinstest10
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  // mail  to: 'Kimchangkyu99@gmail.com',
                  //       subject: "Deploy Frontend Success",
                  //       body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  // mail  to: 'Kimchangkyu99@gmail.com',
                  //       subject: "Failed Pipelinee",
                  //       body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            // 이 agent는 docker node 최신버전을 가지고 steps을 밟을거다
            agent {
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
            post {
              success {
                  echo 'Successfully Lint Backend'
              }
              failure {
                  echo 'I failed :('
              }
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh """
                docker build . -t server --build-arg env=${PROD}
                """
            }
          }
          // failure가 없으면 하나의 stage가 실패해도 다음으로 넝머감
          post {
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        // 실제환경은 ecs, 쿠버네티스 업데이트
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'
//                docker rm -f $(docker ps -aq)
            dir ('./server'){
                sh '''
                docker rm -f $(docker ps -aq)
                docker run -p 80:80 -d server
                '''
            }
          }

          post {
            success {
              echo 'Successfully Build Backend'
              // mail  to: 'Kimchangkyu99@gmail.com',
              //       subject: "Deploy Success",
              //       body: "Successfully deployed!"
                  
            }
          }
        }
    }
}
