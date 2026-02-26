# Frontend CI Packaging Repo

이 저장소는 UniBooker 프론트엔드의 **CI/CD 파이프라인 및 컨테이너 패키징(Dockerfile)** 구성을 공개하기 위한 레포입니다.

<br>

## What’s Included

본 저장소는 CI 도구 비교 및 컨테이너 패키징 구성을 중심으로 구성되어 있습니다.

### Branch Strategy

- `jenkins` branch  
  - `Jenkinsfile`  
    - 이미지 빌드 및 Registry 푸시  
    - GitOps 레포 이미지 태그 자동 업데이트  

- `main` branch  
  - `.github/workflows/main.yml`  
    - GitHub Actions 기반 CI 파이프라인  
    - 동일한 빌드/푸시 및 GitOps 업데이트 로직 구현  

### Common Files

- `Dockerfile`  
  - 프론트엔드 컨테이너 이미지 빌드 정의  
  - CI 도구와 무관하게 공통으로 사용됨

<br>

## What’s NOT included
- 실제 서비스 프론트엔드 소스 코드는 포함하지 않습니다.
  - 사유: 서비스 코드와 배포 정의를 분리하고, 포트폴리오에서는 인프라/배포 자동화 설계를 중심으로 공개하기 위함입니다.

<br>

## Related Repositories

| Repository | Description | Tech Stack |
| --- | --- | --- |
| **[Frontend](https://github.com/somminn/project-a-frontend)** |  **(현재)** Frontend CI packaging repo | Jenkins / Gihub Action  |
| **[Backend](https://github.com/somminn/project-a-backend)** | Backend CI packaging repo | Jenkins / Gihub Action |
| **[Automation](https://github.com/somminn/infra-automation)** | 자동화 스크립트 | Ansible |
| **[GitOps](https://github.com/somminn/project-a-gitops)** | K8s 매니페스트 및 배포 설정 관리 | Kustomize |
