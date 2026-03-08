pipeline { // 오타 수정 (pipline -> pipeline)
    agent any // Jenkins의 사용 가능한 모든 에이전트(노드)에서 실행 가능

    tools {
        maven 'maven 3.8.7' // Jenkins 'Global Tool Configuration'에 등록된 Maven 이름
    }

    environment {
        // 배포에 필요한 환경 변수 설정
        DOCKER_IMAGE = "demo-app" // 도커 이미지 이름
        CONTAINER_NAME = "springboot-container" // 도커 컨테이너 이름
        JAR_FILE_NAME = "app.jar" // 복사할 JAR 파일 이름
        PORT = "8081" // 컨테이너와 연결할 호스트 포트
        REMOTE_USER = "ubuntu" // 원격 인스턴스 서버 사용자명
        REMOTE_HOST = "54.252.148.88" // 원격 인스턴스 IP
        REMOTE_DIR = "/home/ubuntu/deploy" // 원격 서버에서 작업할 경로
        SSH_CREDENTIALS_ID = "939d2c9e-9974-4b24-a158-b1af50af48d8" // 문법 수정 (- 를 = 로 변경)
    }

    // 파이프라인의 여러 단계를 그룹화
    stages {
        
        // stage: 전체 프로세스 중 하나의 큰 단계
        stage('Git checkout') {
            // steps: stage 안에서 실행될 실제 작업 명령어들
            steps {
                // Jenkins가 연결된 Git 저장소에서 최신 코드 체크아웃
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                // Maven을 사용하여 프로젝트 빌드 (이전 단계의 주석이 복사된 것 수정)
                // 테스트는 건너뛰고 패키징 수행 (리눅스 쉘 명령어 실행)
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Prepare Jar') {
            steps {
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }
        stage('Copy to Remote Server') {
            steps {
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]){
                    sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"mkdir -p ${REMOTE_DIR}\""
                    sh "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${JAR_FILE_NAME} Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
                }
            }
        }

        stage('Remote Docker Bulid & Deploy'){
            steps{
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} << ENDSSH
    cd ${REMOTE_DIR} || exit 1
    docker rm -f ${CONTAINER_NAME} || true
    docker build -t ${DOCKER_IMAGE} .
    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}
ENDSSH
                    """
                }
            }
        }

    } // stages 닫는 괄호 추가
} // pipeline 닫는 괄호 추가
