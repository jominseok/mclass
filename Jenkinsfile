pipeline {
    agent any // Jenkins의 사용 가능한 모든 노드에서 실행 가능

    tools {
        maven 'maven 3.8.7'
    }

    environment {
        // 원격 서버 전송에 필요했던 IP, 계정, SSH 키 등의 변수는 모두 삭제했습니다.
        DOCKER_IMAGE = "demo-app" // 도커 이미지 이름
        CONTAINER_NAME = "springboot-container" // 도커 컨테이너 이름
        JAR_FILE_NAME = "app.jar" // 복사할 JAR 파일 이름
        PORT = "8081" // 컨테이너와 연결할 포트
    }

    stages {
        stage('Git checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                // Windows 환경 스크립트 실행 명령어(bat) 사용
                bat 'mvn clean package -DskipTests'
            }
        }
        
        stage('Prepare Jar') {
            steps {
                // Windows 파일 복사 명령어(copy) 사용 및 경로 슬래시(\) 적용
                bat "copy target\\demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}"
            }
        }
        
        stage('Local Docker Build & Deploy') {
            steps {
                // Jenkins 서버 내부에서 바로 도커 빌드 및 실행 진행
                
                // 1. 기존 컨테이너가 있으면 삭제 (없어도 에러 없이 넘어가도록 || echo 처리)
                bat "docker rm -f ${CONTAINER_NAME} || echo No existing container to remove"
                
                // 2. 도커 이미지 빌드
                bat "docker build -t ${DOCKER_IMAGE} ."
                
                // 3. 도커 컨테이너 실행
                bat "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}"
            }
        }
    }
}