# Docker Setup

> 서로 다른 웹 애플리케이션을 **하나의 Docker Compose 환경**에서 실행하기 위해 구성한 개인 개발 인프라 프로젝트입니다.
>
> 개별 애플리케이션 기능 구현을 넘어, 여러 서비스를 연결하는 실행 환경을 직접 구성하고 관리하는 경험을 기록합니다.

Nginx를 외부 요청의 진입점으로 두고, PHP-FPM, Node.js, MySQL, Redis를 각각 독립된 컨테이너로 관리합니다. 도메인에 따라 Laravel 애플리케이션 또는 프런트엔드 개발 서버로 요청을 전달하는 구조를 목표로 했습니다.

## Docker Compose 구성

| 서비스      | 역할                                          |
| ----------- | --------------------------------------------- |
| **Nginx**   | 도메인별 요청 분기, FastCGI 및 리버스 프록시  |
| **PHP-FPM** | Laravel 등 PHP 애플리케이션 실행              |
| **Node.js** | 프런트엔드 개발 서버 및 운영환경 웹 소켓 서버 |
| **MySQL**   | 데이터베이스                                  |
| **Redis**   | 데이터 캐싱                                   |

## 서비스 접속 흐름

```text
Browser (HTTP :80)
       |
       v
    [ Nginx ]
       |
       +-- laravel.wderfer6790.kr -> [ PHP-FPM :9000 ]
       |                                    |

        [ PHP applications ] --- [ MySQL / Redis ]
              (Docker 내부 네트워크 `net`)
```

- `laravel.wderfer6790.kr` → `/var/git/laravel.wderfer6790.kr/public` → PHP-FPM
- PHP, Node.js, MySQL, Redis에는 호스트 포트를 직접 공개하지 않고 Docker의 `net` 네트워크를 사용합니다.
- Nginx 서비스에 `80:80`, `443:443` 포트 매핑이 있지만 **현재 Nginx 설정은 80번 포트만 수신**합니다. HTTPS 인증서 및 443 리스너는 별도 구성해야 합니다.

## 저장소 구조

```text
docker_setup/
├── .gitignore
└── board_by_laravel/
    ├── .env.sample
    ├── docker-compose.yml
    ├── nginx/
    │   ├── Dockerfile
    │   └── default.conf
    ├── php/
    │   └── Dockerfile
    └── node/
        └── Dockerfile
```

애플리케이션 소스 코드는 이 저장소에 포함하지 않고, `SOURCE_ROOT`를 통해 별도 디렉터리를 `/var/git`으로 바인드 마운트합니다.

## 실행 방법

### 1. 저장소 복제 및 환경 변수 설정

```bash
git clone https://github.com/wderfer6790/docker_setup.git
cd docker_setup/board_by_laravel
cp .env.sample .env
```

`.env` 파일에 소스 경로와 MySQL 초기 설정값을 입력합니다.

```dotenv
SOURCE_ROOT=/home/your-user/git

MYSQL_ROOT_PASSWORD=change-me-root
MYSQL_DATABASE=portfolio
MYSQL_USER=portfolio
MYSQL_PASSWORD=change-me-user
```

### 3. 컨테이너 빌드 및 실행

```bash
docker compose up -d --build
docker compose ps
```

### 4. PHP 애플리케이션 준비

필요에 따라 PHP 컨테이너에 접속한 뒤 프로젝트별 설치 명령을 수행합니다.

```bash
docker compose exec php sh
cd /var/git/laravel.wderfer6790.kr
composer install
```

Laravel의 `.env`, `APP_KEY`, 데이터베이스 연결, 마이그레이션 및 디렉터리 권한 등은 **각 애플리케이션 저장소에서 별도로 설정**합니다. Docker 내부에서 MySQL과 Redis의 서비스 호스트명은 각각 `mysql`, `redis`입니다.

### 5. 로컬 도메인 접속

로컬 PC의 `hosts` 파일에 아래 항목을 설정하면 HTTP로 테스트할 수 있습니다.

```text
127.0.0.1 laravel.wderfer6790.kr
```

실제 인터넷 도메인 연결, DNS 및 HTTPS 인증서 발급은 이 저장소의 설정 범위에 포함되지 않습니다.

## 자주 사용하는 명령어

```bash
# 컨테이너 실행 / 중지
docker compose up -d
docker compose down

# 상태 및 로그
docker compose ps
docker compose logs -f nginx
docker compose logs -f php

# 설정 수정 후 재시작
docker compose up -d --build

# 컨테이너 접속
docker compose exec php sh
docker compose exec node sh

# Compose 설정 검사
docker compose config
```

## 연관 프로젝트

- [Board by Laravel](https://github.com/wderfer6790/board_by_laravel) — Laravel 기반 게시판 애플리케이션

---
