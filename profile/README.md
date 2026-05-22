# Cowork Backend

학생회, 동아리, 운영진처럼 여러 사람이 함께 조직을 운영할 때 필요한 업무를 한곳에서 관리하는 백엔드 API 서버입니다.

행사, 예산, 학생 명부, 파일, 회의록, 설문, 일정 조율, 자산 대여처럼 운영 과정에서 자주 흩어지는 정보를 `조직 -> 기수 -> 기능별 데이터` 구조로 정리합니다.

## 1. 프로젝트 소개

Cowork는 단체 운영진을 위한 협업 관리 플랫폼입니다.

운영진은 하나의 조직에 속하고, 조직 안에서 연도/기수 단위의 코호트를 만들어 데이터를 관리합니다. 각 코호트에는 학생 명단, 행사, 회의록, 파일, 예산, 설문, 일정 조율표, 자산 대여 기록 등이 연결됩니다.

이 백엔드 서버는 다음 역할을 담당합니다.

- JWT 기반 회원가입, 로그인, SSO 온보딩, 쿠키 인증 처리
- 조직, 기수, 구성원, 부서 관리
- 행사, 사진, 파일, 회의록, 메모 관리
- 학생 명부 및 회비 납부 상태 관리
- 예산 지출, 영수증 OCR, 통장 내역 매칭 지원
- 설문 생성, 응답 수집, 결과 집계
- 일정 조율표 생성, 응답 수집, 가능 시간 집계
- 자산 등록, 대여, 반납 이력 관리
- Swagger 기반 API 문서 제공

## 2. 문제 정의 및 개발 목적

학생회나 운영진 업무는 보통 카카오톡, 구글폼, 엑셀, 드라이브, 개인 메모에 나뉘어 관리됩니다. 이 방식은 처음에는 편하지만, 시간이 지나면 아래 문제가 생깁니다.

- 행사별 예산, 회의록, 사진, 설문, 파일이 서로 연결되지 않음
- 회비 납부 여부나 자산 대여 상태를 매번 수동으로 확인해야 함
- 기수가 바뀔 때 기존 자료를 이어받기 어렵고 인수인계가 불안정함
- 영수증, 통장 내역, 지출 기록을 사람이 직접 맞춰야 해서 실수가 잦음
- 공개 설문/일정 응답과 관리자용 기능의 권한 구분이 필요함

이 프로젝트의 목적은 운영진의 반복 업무를 줄이고, 조직 운영 데이터를 기수 단위로 안정적으로 남기는 것입니다.

특히 단순 CRUD만 제공하는 것이 아니라, 행사와 관련된 설문·일정·지출·파일을 함께 묶고, OCR/엑셀 업로드/Redis 큐/S3 저장소처럼 실제 운영에 필요한 기능까지 고려했습니다.

## 3. 주요 기능

### 인증 및 사용자 관리

- 이메일/비밀번호 회원가입 및 로그인
- JWT Access/Refresh 토큰 발급
- 브라우저 클라이언트용 HttpOnly 쿠키 인증
- 숭실대 SSO 기반 임시 토큰 발급 및 회원가입 흐름
- 가입 승인 대기 `PENDING`, 승인 완료 `ACTIVE` 상태 관리
- 약관/개인정보 동의 여부 확인 필터

### 조직, 기수, 부서 관리

- 조직별 초대 코드 관리
- 조직 안에서 여러 기수 `Cohort` 생성
- 기수별 멤버 역할 관리: `ADMIN`, `EDITOR`, `VIEWER`
- 조직별 커스텀 부서명 관리
- 회장단 부서 기본 보장

### 행사 관리

- 행사명, 기간, 장소, 담당 부서, 담당자, 예산 관리
- 행사 상태 관리: `PLANNING`, `ONGOING`, `DONE`, `CANCELLED`
- 행사 사진 업로드 및 삭제
- 행사에 설문, 일정 조율표, 지출, 파일, 회의록 연결 가능

### 예산 및 영수증 관리

- 지출 내역 등록, 수정, 삭제, 조회
- 기간, 부서, 카테고리, 행사 기준 필터링
- 지출 요약 통계 제공
- 영수증 파일 업로드
- CLOVA OCR을 이용한 영수증 정보 추출
- 통장/엑셀 내역 파싱 및 영수증 결제 시각 기반 매칭
- 모바일 세션을 통한 외부 영수증 업로드 흐름 지원

### 학생 명부 및 회비 관리

- 학생 개별 등록, 수정, 삭제
- 엑셀 파일 기반 학생 명부 일괄 등록
- 회비 납부 상태 `PAID`, `UNPAID` 관리
- 회비 납부 일괄 처리
- 기수별 학생 수, 납부자 수, 미납자 수 요약 제공

### 파일 및 회의록 관리

- 폴더/파일 트리 구조 관리
- 파일 업로드, 다운로드, 이름 변경, 이동, 삭제
- 파일 작업 로그 저장
- 워크스페이스별 회의록 작성
- 회의록 참석자, 안건, 내용, 첨부파일 관리

### 설문 관리

- 설문 생성, 수정, 삭제
- 설문 상태 관리: `DRAFT`, `OPEN`, `CLOSED`
- 단답형, 장문형, 객관식, 체크박스, 드롭다운 질문 지원
- 공개 응답 제출 API 제공
- 설문 결과 집계
- Redis Stream 기반 비동기 응답 처리 옵션 지원

### 일정 조율

- 날짜 범위, 시간 범위, 슬롯 단위로 일정 조율표 생성
- 참여자별 가능 시간 제출
- 공개 응답 제출 API 제공
- 가장 많이 겹치는 시간대 집계
- 행사 또는 회의 일정 조율에 연결 가능

### 자산 대여 관리

- 자산명, 수량, 위치, 상태, 사진 관리
- 대여 가능 수량 자동 계산
- 대여자, 학번, 담당자, 신분증 제출 여부, 반납 예정일 기록
- 반납 처리 및 대여 이력 관리

### 감사 로그

- 주요 데이터 변경 이력 저장
- 작업자, 작업 유형, 대상, 변경 전/후 데이터 추적
- 최근 변경 내역 조회

## 4. 시스템 아키텍처

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/7cba4cf7-2d6c-4967-b623-8d0a101b27c2" />

### 패키지 구성

| 패키지 | 역할 |
| --- | --- |
| `auth` | 로그인, 회원가입, JWT, SSO, 쿠키 인증 |
| `user` | 사용자 계정 및 Spring Security 사용자 로딩 |
| `organization` | 조직 및 조직별 부서 관리 |
| `cohort` | 기수, 기수 멤버, 역할 관리 |
| `event` | 행사 및 행사 사진 관리 |
| `budget` | 지출, 영수증 OCR, 통장 매칭 |
| `student` | 학생 명부 및 회비 납부 관리 |
| `file` | 파일/폴더, 파일 작업 로그 |
| `workspace` | 워크스페이스, 회의록, 회의 첨부파일 |
| `survey` | 설문, 질문, 선택지, 응답, 응답 큐 |
| `schedule` | 일정 조율표, 참여자, 시간 응답 |
| `asset` | 자산, 대여/반납 기록 |
| `mobile` | 모바일 영수증 업로드 세션 |
| `audit` | 감사 로그 |
| `common` | 공통 응답, 예외, 파일 저장소 |
| `config` | Security, Swagger, JPA, Web 설정 |

## 5. ERD

아래 ERD는 핵심 테이블 중심으로 정리한 구조입니다.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/70582197-f25f-4673-9b32-f50eca9cbb58" />


### 핵심 테이블 설명

| 테이블 | 설명 |
| --- | --- |
| `organizations` | 서비스 최상위 단위. 조직명, 학과, 초대 코드를 저장 |
| `users` | 로그인 계정. 조직에 소속되며 가입 승인 상태를 가짐 |
| `cohorts` | 조직 안의 기수/연도 단위 그룹 |
| `cohort_members` | 사용자가 특정 기수에서 갖는 역할과 부서 |
| `cowork_events` | 행사 정보. 다른 기능과 연결되는 중심 데이터 |
| `expenses` | 지출 내역. 영수증, 행사, 부서, 결제 시각 관리 |
| `students` | 기수별 학생 명부와 회비 납부 상태 |
| `file_items` | 파일/폴더 트리 구조 |
| `workspaces` | 부서별 또는 공용 회의록 공간 |
| `surveys` | 설문 기본 정보 |
| `survey_questions` | 설문 질문 |
| `survey_responses` | 설문 제출 단위 |
| `timetables` | 일정 조율표 |
| `assets` | 자산/비품 정보 |
| `rental_records` | 자산 대여 및 반납 이력 |
| `audit_logs` | 주요 변경 이력 |
| `stored_files` | 로컬/S3에 저장된 파일 메타데이터 |

## 6. DB 설계 의도

### 조직과 기수를 기준으로 데이터 분리

대부분의 운영 데이터는 `cohort_id`를 가집니다. 같은 조직이라도 기수가 바뀌면 행사, 학생, 예산, 파일이 달라지기 때문에 기수 단위로 데이터를 분리했습니다.

```text
Organization
  -> Cohort
      -> Event / Student / Expense / File / Survey / Timetable / Asset
```

이 구조 덕분에 이전 기수 자료는 보존하면서, 새 기수는 독립적으로 운영할 수 있습니다.

### 사용자와 학생 명부를 분리

`users`는 로그인 가능한 운영진 계정이고, `students`는 관리 대상 학생 명부입니다.

두 개를 분리한 이유는 학생 명부에 있는 모든 사람이 로그인 계정을 가질 필요는 없기 때문입니다. 예를 들어 회비 납부 관리 대상 학생은 `students`에만 있어도 되고, 운영진으로 로그인하는 사람만 `users`에 존재하면 됩니다.

### 행사 중심 연결 구조

행사는 운영 업무의 중심이 되는 데이터입니다. 그래서 `event_id`를 여러 테이블에서 선택적으로 사용합니다.

- 행사 지출: `expenses.event_id`
- 행사 설문: `surveys.event_id`
- 행사 일정 조율: `timetables.event_id`
- 행사 파일: `file_items.event_id`
- 행사 회의록: `meetings.event_id`
- 행사 사진: `event_photos.event_id`

이렇게 하면 행사 상세 화면에서 관련 자료를 한 번에 보여줄 수 있습니다.

### 파일은 실제 파일과 메타데이터를 분리

파일 자체는 로컬 디스크 또는 S3에 저장하고, DB에는 경로와 메타데이터만 저장합니다.

- `file_items`: 사용자가 보는 파일/폴더 구조
- `stored_files`: 저장소 타입, bucket, object key, 원본 파일명, 크기 등 저장 메타데이터
- `file_logs`: 파일 업로드/이름변경/이동/삭제 이력

이 구조는 저장소를 로컬에서 S3로 바꾸더라도 서비스 로직을 크게 바꾸지 않도록 설계된 방식입니다.

### 공개 응답 API와 관리자 API 구분

설문 응답, 일정 조율 응답, 모바일 영수증 업로드는 외부 참여자도 접근할 수 있어야 합니다. 그래서 일부 API는 인증 없이 접근할 수 있도록 열어두고, 그 외 관리 기능은 JWT 인증을 요구합니다.

### JSON 컬럼 사용

선택지가 여러 개인 값이나 유동적인 상세 정보는 JSON 컬럼으로 저장합니다.

- 행사 담당자 목록: `cowork_events.organizers`
- 자산 태그: `assets.tags`
- 설문 선택 답변: `response_answers.selected_option_ids`
- 일정 선택 슬롯: `timetable_responses.selected_slots`
- 파일 로그 상세: `file_logs.detail`

고정된 컬럼으로 만들기 어려운 목록형 데이터를 단순하게 저장하기 위한 선택입니다.

### Soft Delete 적용

여러 주요 테이블은 `deleted_at`을 사용합니다. 실제 데이터를 바로 지우지 않고 삭제 시각을 남겨 복구 가능성과 이력 추적 가능성을 확보합니다.

예: `events`, `expenses`, `students`, `files`, `assets`, `surveys`, `timetables`

## 7. 기술 스택

| 구분 | 기술 |
| --- | --- |
| Language | Java 17 |
| Framework | Spring Boot 3.4.4 |
| Web | Spring MVC |
| Security | Spring Security, JWT, HttpOnly Cookie |
| ORM | Spring Data JPA, Hibernate |
| Database | MySQL 8 |
| Migration | Flyway |
| Cache / Queue | Redis, Redis Stream |
| File Storage |  AWS S3 |
| OCR | CLOVA OCR |
| Excel / CSV | Apache POI, OpenCSV |
| API Docs | Springdoc OpenAPI, Swagger UI |
| Build | Gradle |
| Deploy | Docker, Docker Compose, Caddy |

## 8. API 문서

Swagger UI는 서버 실행 후 아래 주소에서 확인할 수 있습니다.

```text
http://localhost:8080/swagger-ui/index.html
```

OpenAPI JSON은 아래 주소에서 확인할 수 있습니다.

```text
http://localhost:8080/v3/api-docs
```

### 공통 응답 형식

성공 응답:

```json
{
  "success": true,
  "data": {},
  "message": null,
  "code": null
}
```

에러 응답:

```json
{
  "success": false,
  "data": null,
  "message": "에러 메시지",
  "code": "ERROR_CODE"
}
```

### 인증 방식

로그인 또는 회원가입에 성공하면 Access Token과 Refresh Token이 발급됩니다. 브라우저 클라이언트에서는 HttpOnly 쿠키를 사용하도록 설계되어 있습니다.

- Access Cookie: `cowork_access_token`
- Refresh Cookie: `cowork_refresh_token`
- Access Token 만료 시: `POST /api/auth/refresh`
- 로그아웃 시: `POST /api/auth/logout`

Swagger에서는 레거시 호환을 위해 Bearer Token 인증도 함께 문서화되어 있습니다.

### 주요 API 목록
https://github.com/CoWork-Service/Back/blob/main/docs/api-list.md
