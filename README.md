# Spring PetClinic CI/CD Pipeline

이 저장소는 DevSecOps 구현을 위해 [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) 프로젝트를 포크한 저장소입니다. 구현한 파이프라인은 빌드, 배포 프로세스에 보안을 추가해 개발 속도와 보안 수준을 동시에 높이는 목표를 갖고 있습니다.

## 프로젝트 소개
Spring PetClinic은 Spring Boot를 사용하여 간단한 웹 애플리케이션을 구축하는 데 초점을 맞춘 데모 프로젝트입니다. 이 포크된 버전에는 다음 단계를 자동화하는 CI/CD 파이프라인이 포함되어 있습니다:

- **소스 코드 관리:** GitHub에서 저장소를 클론합니다.
- **빌드 및 테스트:** Maven을 사용하여 프로젝트를 빌드하고 테스트를 실행합니다.
- **코드 품질 분석:** SonarQube 분석을 실행하고 품질 게이트를 강제합니다.
- **보안 스캔:** OWASP Dependency Check 및 Trivy를 사용하여 정적 및 동적 보안 분석을 수행합니다.
- **아티팩트 배포:** 빌드된 아티팩트를 Nexus에 업로드합니다.
- **Dockerization:** Docker 이미지를 빌드하고 컨테이너 레지스트리에 푸시합니다.
- **배포:** Kubernetes 배포 매니페스트를 업데이트하고 변경 사항을 GitHub에 푸시합니다.
- **알림:** Slack에 빌드 알림을 전송합니다.

## 파이프라인 단계

1. **Slack Send Channel**
    
    빌드 시작 시 지정된 Slack 채널로 알림을 전송합니다.

2. **Clean Up**

    빌드를 위한 깨끗한 환경을 보장하기 위해 작업 공간을 정리합니다.

3. **Checkout from Git**

    GitHub URL과 브랜치에서 저장소를 클론합니다.

4. **Build**
    
    Maven을 사용하여 프로젝트를 빌드하고 유닛 테스트를 실행합니다.

5. **SonarQube Analysis**

    코드 품질 및 보안 취약점을 분석합니다.

6. **Quality Gate**

    SonarQube 품질 게이트를 강제하여 코드 품질 표준을 충족하도록 합니다.

7. **OWASP ZAP**

    의존성 스캔을 수행하여 취약점을 식별합니다.

8. **Trivy File System Scan**

    프로젝트 디렉터리를 스캔하여 보안 문제를 확인합니다.

9. **Nexus Artifact Upload**

    빌드된 JAR 파일을 Nexus 저장소에 업로드합니다.

10. **Docker Build**

    Spring Boot `build-image` 플러그인을 사용하여 Docker 이미지를 빌드합니다.

11. **Docker Build & Push**

    Docker 이미지를 태그하고 지정된 컨테이너 레지스트리에 푸시합니다.

12. **Trivy Image Scan**

    Docker 이미지를 스캔하여 취약점을 확인합니다.

13. **Update the Deployment Tags**

    새로운 Docker 이미지 태그로 Kubernetes 배포 매니페스트를 업데이트합니다.

14. **Push the Changed Deployment File to Git**

    업데이트된 Kubernetes 매니페스트를 GitHub에 커밋하고 푸시합니다.

## 환경 변수

| 변수               | 설명                                         |
|-------------------|---------------------------------------------|
| `APP_NAME`        | 애플리케이션 이름 (예: "account").          |
| `RELEASE`         | 릴리즈 버전 (예: "1.0.0").                |
| `IMAGE_TAG`       | Docker 이미지 태그 (예: "1.0.0-123").      |
| `DOCKER_REGISTRY` | Docker 레지스트리 사용자명.                 |
| `DOCKER_REPOSITORY` | Docker 저장소 이름.                        |
| `JENKINS_API_TOKEN` | Jenkins API 토큰.                          |

## 요구 사항

- 다음 플러그인이 설치된 Jenkins:
  - Pipeline
  - Slack Notification
  - SonarQube Scanner
  - Nexus Artifact Uploader
  - OWASP Dependency Check
- Maven 3.x 및 JDK 17 설치.
- Docker 및 Docker Registry 자격 증명 설정.
- Kubernetes 클러스터 배포.
- Kubernetes 매니페스트가 포함된 GitHub 저장소.
- Slack 작업 공간 및 API 토큰.

## 파이프라인 실행 방법

1. 필요한 플러그인 및 도구로 Jenkins를 설정합니다.
2. Jenkins에 필요한 자격 증명을 구성합니다:
   - GitHub 자격 증명 (`git-credential`).
   - Docker Registry 자격 증명 (`docker-credential`).
   - SonarQube 토큰 (`sonarqube-token`).
   - Slack 토큰 (`slack-credential`).
3. 이 저장소를 Jenkins에 클론합니다.
4. 파이프라인을 실행합니다.

## 알림
파이프라인은 다음 단계에서 Slack 알림을 전송합니다:

- **시작:** 빌드 시작 알림과 Jenkins 작업 링크를 전송합니다.
- **성공:** 빌드 성공 시 알림을 전송합니다.
- **실패:** 빌드 실패 시 알림을 전송합니다.

## 저장소 구조

```plaintext
├── ...
├── Jenkinsfile                  # Jenkins 파이프라인 스크립트
├── k8s                          # Kubernetes 배포 매니페스트
├── pom.xml                      # Maven 프로젝트 구성 파일
├── src                          # Spring PetClinic 애플리케이션 소스 코드
├── ...
```
