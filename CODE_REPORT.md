# EKS Example 프로젝트 코드 분석 보고서

## 목차

1. [개요](#개요)
2. [프로젝트 구조](#프로젝트-구조)
3. [상세 분석](#상세-분석)
   - 3.1. [Dockerfile 분석](#dockerfile-분석)
   - 3.2. [Nginx 설정 파일](#nginx-설정-파일)
   - 3.3. [웹 페이지 구성](#웹-페이지-구성)
4. [기술 스택](#기술-스택)
5. [주요 기능](#주요-기능)
6. [배포 환경](#배포-환경)
7. [결론 및 권장사항](#결론-및-권장사항)
8. [작성자 정보](#작성자-정보)

---

## 개요

본 프로젝트는 **Amazon EKS (Elastic Kubernetes Service) 실습용 애플리케이션**입니다. Nginx 웹 서버를 기반으로 한 간단한 "Hello World" 웹 애플리케이션으로, 컨테이너화 및 Kubernetes 배포 실습을 위해 설계되었습니다.

**프로젝트명:** eks-example
**용도:** AWS Training and Certification - Lab 3
**버전:** Version 2

## 프로젝트 구조

```
eks-example/
├── .claude/              # Claude Code 스킬 디렉토리
│   └── skills/
│       ├── pdf-report.md
│       └── README.md
├── .git/                 # Git 저장소
├── src/                  # 소스 코드 디렉토리
│   ├── hello.conf       # Nginx 설정 파일
│   └── index.html       # 메인 HTML 페이지
├── Dockerfile           # 컨테이너 이미지 빌드 파일
└── README.md            # 프로젝트 문서
```

## 상세 분석

### Dockerfile 분석

**파일:** `Dockerfile`

```dockerfile
FROM 132541598664.dkr.ecr.us-west-2.amazonaws.com/awstc:eks-ilt-lab3
RUN rm /etc/nginx/conf.d/*
ADD src/hello.conf /etc/nginx/conf.d/
ADD src/index.html /usr/share/nginx/html/
```

**주요 특징:**

1. **베이스 이미지**
   - AWS Training and Certification 공식 이미지 사용
   - ECR (Elastic Container Registry) 저장소: `us-west-2` 리전
   - 이미지: `eks-ilt-lab3` (EKS 실습용 사전 구성 이미지)

2. **구성 초기화**
   - 기존 Nginx 설정 파일을 모두 제거 (`/etc/nginx/conf.d/*`)
   - 깨끗한 상태에서 커스텀 설정 적용

3. **파일 배포**
   - 커스텀 Nginx 설정: `hello.conf`
   - HTML 웹 페이지: `index.html`

### Nginx 설정 파일

**파일:** `src/hello.conf`

```nginx
server {
    listen 80;

    root /usr/share/nginx/html;
    try_files /index.html =404;

    expires -1;

    sub_filter_once off;
    sub_filter 'server_hostname' '$hostname';
    sub_filter 'server_address' '$server_addr:$server_port';
    sub_filter 'server_url' '$request_uri';
    sub_filter 'server_date' '$time_local';
    sub_filter 'request_id' '$request_id';
}
```

**설정 분석:**

| 설정 항목 | 설명 | 목적 |
|---------|------|------|
| `listen 80` | HTTP 포트 80에서 수신 | 표준 웹 서비스 포트 |
| `root /usr/share/nginx/html` | 웹 루트 디렉토리 | HTML 파일 위치 지정 |
| `try_files /index.html =404` | 모든 요청을 index.html로 라우팅 | SPA (Single Page App) 동작 |
| `expires -1` | 캐시 비활성화 | 항상 최신 컨텐츠 제공 |
| `sub_filter` | 동적 텍스트 치환 | 서버 정보를 실시간으로 주입 |

**동적 변수 치환:**

- `server_hostname` → 실제 호스트명 (Pod 이름)
- `server_address` → 서버 IP 주소 및 포트
- `server_url` → 요청 URI
- `server_date` → 현재 시간
- `request_id` → 고유 요청 ID

이 기능은 **Kubernetes 환경에서 각 Pod의 정보를 확인**하는 데 유용합니다.

### 웹 페이지 구성

**파일:** `src/index.html`

```html
<head>
<title>Hello World Version 2</title>
</head>
<div class="info">
<h>Hello World!</h>
<p><span>Server&nbsp;address:</span> <span>server_address</span></p>
<p><span>Server&nbsp;name:</span> <span>server_hostname</span></p>
<p class="smaller"><span>Date:</span> <span>server_date</span></p>
<p class="smaller"><span>URI:</span> <span>server_url</span></p>
</div>
```

**페이지 특징:**

- **제목:** "Hello World Version 2"
- **표시 정보:**
  - 서버 주소 (IP:Port)
  - 서버 호스트명 (Pod ID)
  - 요청 시간
  - 요청 URI

**플레이스홀더 변수:**
HTML의 텍스트(예: `server_address`)는 Nginx의 `sub_filter` 지시어에 의해 실시간으로 치환됩니다.

## 기술 스택

| 계층 | 기술 | 버전/설명 |
|------|------|-----------|
| **컨테이너** | Docker | Dockerfile 기반 이미지 빌드 |
| **웹 서버** | Nginx | 경량 고성능 웹 서버 |
| **오케스트레이션** | Kubernetes (EKS) | AWS 관리형 Kubernetes |
| **레지스트리** | Amazon ECR | us-west-2 리전 |
| **프론트엔드** | HTML/CSS | 정적 웹 페이지 |

## 주요 기능

### 1. **Pod 식별 기능**
각 Kubernetes Pod가 자신의 호스트명을 표시하여 로드밸런싱 동작을 시각적으로 확인할 수 있습니다.

### 2. **실시간 서버 정보 표시**
- 서버 IP 주소
- 요청 시간
- 고유 요청 ID

### 3. **캐시 비활성화**
`expires -1` 설정으로 항상 최신 정보를 표시합니다.

### 4. **간단한 배포**
최소한의 구성으로 빠른 배포 및 스케일링이 가능합니다.

## 배포 환경

### Docker 이미지 빌드

```bash
# 이미지 빌드
docker build -t eks-example:v2 .

# ECR에 푸시
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 132541598664.dkr.ecr.us-west-2.amazonaws.com
docker tag eks-example:v2 132541598664.dkr.ecr.us-west-2.amazonaws.com/eks-example:v2
docker push 132541598664.dkr.ecr.us-west-2.amazonaws.com/eks-example:v2
```

### Kubernetes 배포

```yaml
# deployment.yaml 예시
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: 132541598664.dkr.ecr.us-west-2.amazonaws.com/eks-example:v2
        ports:
        - containerPort: 80
```

### Service 구성

```yaml
# service.yaml 예시
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
spec:
  type: LoadBalancer
  selector:
    app: hello-world
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

## 결론 및 권장사항

### 현재 상태 평가

**강점:**
- ✅ 명확하고 간단한 구조
- ✅ Kubernetes 학습에 최적화
- ✅ 동적 서버 정보 표시로 Pod 식별 용이
- ✅ AWS EKS 환경에 특화된 베이스 이미지

**개선 가능 영역:**
- ⚠️ HTML 구조 불완전 (DOCTYPE, body 태그 누락)
- ⚠️ CSS 스타일이 정의되지 않음
- ⚠️ 프로덕션 환경을 위한 보안 설정 부재

### 권장사항

#### 1. HTML 구조 개선
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello World Version 2</title>
    <style>
        /* CSS 스타일 추가 */
    </style>
</head>
<body>
    <div class="info">
        <!-- 컨텐츠 -->
    </div>
</body>
</html>
```

#### 2. 헬스체크 엔드포인트 추가
```nginx
location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
}
```

#### 3. 보안 헤더 추가
```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
```

#### 4. 로깅 개선
```nginx
access_log /var/log/nginx/access.log combined;
error_log /var/log/nginx/error.log warn;
```

#### 5. 멀티스테이지 빌드 (선택사항)
더 작은 이미지 크기와 보안을 위해 멀티스테이지 빌드를 고려할 수 있습니다.

### 학습 활용 방안

본 프로젝트는 다음 개념을 학습하는 데 활용할 수 있습니다:

1. **컨테이너화:** Docker 이미지 빌드 및 실행
2. **Kubernetes 기본:** Deployment, Service, ReplicaSet
3. **로드밸런싱:** 여러 Pod 간 트래픽 분산 확인
4. **스케일링:** Horizontal Pod Autoscaling
5. **AWS 통합:** ECR, EKS, Load Balancer 연동

---

## 작성자 정보

**작성일:** 2025-10-23
**작성자:** Claude Code
**프로젝트:** eks-example 코드 분석

![서명](./signature.png)

---
*본 문서는 Claude Code PDF Report Writer로 작성되었습니다.*
