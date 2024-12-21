# Jenkins DevSecOps CI/CD 파이프라인
이 프로젝트는 Jenkins를 사용하여 **DevSecOps** CI/CD 파이프라인을 구현했습니다. 이 파이프라인은 다음의 기능을 제공합니다:

## **기능**
- Git에서 코드 체크아웃
- Maven 빌드 및 테스트
- SonarQube를 통한 코드 품질 분석
- OWASP ZAP을 통한 보안 취약점 스캔
- Nexus에 아티팩트 업로드
- Docker 이미지 빌드 및 푸시
- Trivy 이미지 스캔
- Kubernetes 배포 파일 업데이트
- Git에 변경 사항 푸시

## **파이프라인 구성**
이 파이프라인은 다음의 단계들로 구성됩니다:

1. **Slack 알림**: 파이프라인 실행 시작 시 Slack 채널에 알림 전송
2. **Git 체크아웃**: Git에서 최신 코드 가져오기
3. **Maven 빌드**: 프로젝트 빌드 및 테스트
4. **SonarQube 분석**: 코드 품질 분석
5. **품질 게이트**: SonarQube 품질 기준 체크
6. **OWASP ZAP 스캔**: 보안 취약점 검사
7. **Nexus 아티팩트 업로드**: 빌드된 아티팩트 Nexus에 업로드
8. **Docker 빌드 및 푸시**: Docker 이미지 빌드 및 AWS ECR에 푸시
9. **Trivy 이미지 보안 스캔**: Docker 이미지 보안 스캔
10. **배포 파일 업데이트**: Kubernetes 배포 파일에 새로운 이미지 태그 반영
11. **Git 푸시**: 변경된 배포 파일을 Git에 푸시

## **배포 환경**
- **AWS ECR**: Docker 이미지를 저장하고 배포
- **SonarQube**: 코드 품질 분석
- **OWASP ZAP**: 보안 취약점 검사

## **필요한 도구들**
- Jenkins
- Maven
- Docker
- AWS CLI
- Trivy
- SonarQube
- Nexus Repository
- ArgoCD
- AWS S3, ECR, EKS

## **설치 방법**
1. Jenkins 서버 설치
2. GitHub 리포지토리 연결
3. SonarQube, AWS, Nexus와 연결
4. 필요한 Jenkins 플러그인 설치
5. `jenkinsfile`을 Jenkins에 추가하여 파이프라인 실행

## **결과**
- 파이프라인이 성공적으로 실행되면, 자동으로 Docker 이미지가 생성되고 배포됩니다.
- SonarQube와 OWASP ZAP을 통해 코드 품질 및 보안 취약점 검사가 수행됩니다.
