# 📊 주간보고서 자동화 시스템

> 슈퍼컴퍼니 서비스개발1팀 — AI가 구글 드라이브 · Gmail 데이터를 수집하여 PPT 주간보고서를 자동 생성하고 팀장에게 발송하는 자동화 워크플로우

---

## ✨ 주요 기능

| 기능 | 설명 |
|------|------|
| 📧 **메일 수집** | Gmail에서 업무 관련 이메일 자동 필터링 |
| 📁 **회의록 수집** | 구글 드라이브 회의록 자동 수집 및 누락 감지 |
| 📝 **회의록 자동 작성** | STT 텍스트 파일로 공식 회의록 생성 후 드라이브 업로드 |
| 📑 **PPT 자동 생성** | 회사 양식 디자인을 유지한 채 내용만 자동 입력 |
| ✅ **컨펌 프로세스** | 발송 전 사용자 확인 (Human-in-the-loop) |
| 📤 **자동 발송** | 컨펌 후 팀장에게 이메일 + PPT 첨부 자동 발송 |

---

## 🗂️ 데이터 소스

```
소스 1 ── Gmail
           └─ fastkjn1@gmail.com 수발신 업무 메일 (최근 1주일)
           └─ 자동발송·광고·스팸 자동 제외

소스 2 ── 구글 드라이브 / 회의록 폴더
           └─ 이번 주 생성·수정된 파일만 수집
           └─ 주요 결정사항, 액션아이템, 다음 주 예정 추출

소스 3 ── 구글 드라이브 / 업무 결과물 폴더
           └─ 이번 주 생성·수정된 파일만 수집
           └─ 완료 업무명, 산출물, 수치·지표 추출

소스 4 ── STT 텍스트 파일 (회의 녹음 변환본)
           └─ D:\report-project\skills\회의록\references\회의녹음\
           └─ *_STT원본.txt → 회의록 자동 작성
```

> 수집 기간: 매 실행일 기준 **직전 월요일 ~ 실행 당일**

---

## 📁 프로젝트 구조

```
report-project/
├── CLAUDE.md                        # AI 자동화 규칙 설정
├── 자동화업무정리.md                  # 워크플로우 설계 문서
├── gcp-oauth.keys.json              # Google API OAuth 키 (비공개)
│
├── week_pptx/
│   ├── ppt_template/
│   │   └── 주간보고서.pptx           # 보고서 양식 (원본 — 수정 금지)
│   └── weekly_report/
│       ├── 주간보고서_YYYYMMDD_v1.pptx  # 생성된 주간보고서
│       └── 발송완료/                  # 발송 완료 보고서 보관
│
└── skills/                          # AI 워크플로우 Skill 모음
    ├── 주간보고서/
    │   ├── SKILL.md                 # 주간보고서 작성 전체 워크플로우
    │   └── references/
    │       ├── slide-mapping.md     # 슬라이드 섹션별 작성 규칙
    │       └── email-template.md   # 이메일 발송 양식
    ├── 회의록/
    │   ├── SKILL.md                 # 회의록 작성 전체 워크플로우
    │   └── references/
    │       ├── meeting-template.md  # 회의록 공식 양식
    │       └── writing-rules.md    # 작성 규칙 상세
    └── pptx/
        ├── SKILL.md                 # PPT 생성·편집 가이드
        ├── editing.md
        ├── pptxgenjs.md
        └── scripts/                 # PPT 처리 유틸리티 스크립트
```

---

## 🤖 AI Skills

AI가 상황에 맞게 아래 Skill을 자동으로 선택·실행합니다.

### 📑 주간보고서 작성 (`skills/주간보고서/SKILL.md`)

**트리거:** "주간보고서 작성해줘", "이번 주 보고서 만들어줘", "보고서 써줘"

```
Step 1 ── Gmail 업무 메일 수집          [자동]
Step 2 ── STT 파일 → 회의록 누락 확인   [자동]
Step 3 ── 구글 드라이브 회의록 수집      [자동]
Step 4 ── 구글 드라이브 업무 결과물 수집 [자동]
Step 5 ── PPT 자동 작성 및 저장         [자동]
Step 6 ── 발송 컨펌 요청               [사용자 확인 ← 유일한 멈춤 지점]
Step 7 ── 팀장에게 이메일 발송          [컨펌 후 실행]
```

### 📝 회의록 작성 (`skills/회의록/SKILL.md`)

**트리거:** "회의록 작성해줘", "STT 파일 정리해줘", "미팅 정리해줘"

```
Step 1 ── STT 파일 목록 확인
Step 2 ── STT 텍스트 분석 및 항목 추출
Step 3 ── 회의록 초안 작성 + 사용자 컨펌
Step 4 ── Google Drive 회의록 폴더에 업로드
```

### 🖼️ PPT 생성·편집 (`skills/pptx/SKILL.md`)

**트리거:** .pptx 파일이 포함된 모든 작업 (생성, 편집, 읽기, 변환 등)

---

## ⚙️ 사전 준비

### 1. MCP 서버 설정 (`.mcp.json`)

```json
{
  "mcpServers": {
    "filesystem": { ... },
    "google-drive": { ... },
    "gmail": { ... }
  }
}
```

### 2. Google OAuth 인증

- Google Cloud Console에서 **OAuth 2.0 클라이언트 ID** (데스크톱 앱) 생성
- **Gmail API · Google Drive API** 활성화 필요
- `gcp-oauth.keys.json` 프로젝트 루트에 위치
- 최초 실행 시 브라우저 인증 → 이후 자동 갱신

---

## 📋 보고서 작성 규칙

- PPT 디자인 · 레이아웃 · 폰트 · 색상 **변경 금지**
- 텍스트 입력만 허용, 슬라이드 추가 · 삭제 **금지**
- 슬라이드당 **3줄 이내** 핵심 내용 요약 (격식체)
- 해당 주에 없는 항목 → **"해당 없음"** 표기
- 동일 내용 여러 소스 중복 시 → **한 번만** 기재

---

## 📧 발송 규칙

| 항목 | 내용 |
|------|------|
| **수신자** | jn.kim \<fastkjn1@gmail.com\> |
| **발신자** | Jina Kim \<fastkjn1@gmail.com\> |
| **제목 형식** | `[주간보고] YYYY년 MM월 N주차 업무 요약` |
| **발송 조건** | 사용자 컨펌 후에만 발송 (자동 발송 절대 없음) |
| **발송 후 보관** | `week_pptx/weekly_report/발송완료/` |

---

## 🔒 보안 및 주의사항

- 인사 · 급여 · 개인 식별 정보가 포함된 메일은 수집 제외
- 원본 양식 파일 (`week_pptx/ppt_template/주간보고서.pptx`) 덮어쓰기 금지
- `gcp-oauth.keys.json` 은 절대 공개 저장소에 업로드 금지
- 확인 없이 자동 발송하는 경우는 어떠한 상황에서도 없음

---

## 🛠️ 기술 스택

![Python](https://img.shields.io/badge/Python-3.13-blue?style=flat-square)
![Google Drive](https://img.shields.io/badge/Google_Drive-API-4285F4?style=flat-square&logo=googledrive&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-API-EA4335?style=flat-square&logo=gmail&logoColor=white)
![PowerPoint](https://img.shields.io/badge/python--pptx-PPT_생성-B7472A?style=flat-square)
![MCP](https://img.shields.io/badge/MCP-AI_Tools-orange?style=flat-square)

---

<p align="center">
  <sub>슈퍼컴퍼니 서비스개발1팀 · Jina Kim</sub>
</p>
