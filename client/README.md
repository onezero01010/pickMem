# Pick The Moment

무제한으로 원하는 모습을 찍고 꾸미세요! 무료 폴라로이드 스타일 사진 편집 웹 애플리케이션입니다.

## 📋 프로젝트 개요

Pick The Moment는 사용자가 사진을 업로드하고 다양한 프레임, 스티커, 패턴을 적용하여 폴라로이드 스타일의 사진을 만들 수 있는 웹 애플리케이션입니다. Fabric.js를 활용한 이미지 편집 기능과 html2canvas를 통한 결과물 저장 기능을 제공합니다.

## 🚀 기술 스택

### 프레임워크 & 라이브러리
- **Vue.js 2.6.11** - 프론트엔드 프레임워크
- **Vue Router 3.2.0** - 라우팅 관리
- **Vuex 3.4.0** - 상태 관리
- **Fabric.js 5.2.1** - 캔버스 기반 이미지 편집
- **html2canvas 1.4.1** - 캔버스를 이미지로 변환
- **vue-draggable 2.24.3** - 드래그 앤 드롭 기능
- **vue-toasted 1.1.28** - 토스트 알림
- **vue-meta 2.4.0** - 메타 태그 관리

### 빌드 도구
- **Vue CLI 4.5.15** - 프로젝트 스캐폴딩 및 빌드
- **Babel** - JavaScript 트랜스파일링
- **Sass** - CSS 전처리기

### UI 라이브러리
- **Bootstrap** - UI 컴포넌트
- **Material Design Icons** - 아이콘

## 📁 프로젝트 구조

```
client/
├── public/                 # 정적 파일
├── src/
│   ├── assets/            # 리소스 파일
│   │   ├── css/           # 스타일시트
│   │   ├── fonts/         # 폰트 파일
│   │   ├── frame/         # 프레임 이미지 (1_2.png, 2_1.png 등)
│   │   ├── pattern/       # 패턴 이미지 (pattern_1.png ~ pattern_9.png)
│   │   ├── stickers/      # 스티커 이미지 (꽃잎, 귀여운 일러스트 등)
│   │   └── img/           # 기타 이미지
│   ├── components/        # 공통 컴포넌트
│   │   ├── nav.vue        # 네비게이션 바
│   │   ├── footer.vue     # 푸터
│   │   └── modal.vue      # 모달 컴포넌트
│   ├── views/             # 페이지 컴포넌트
│   │   ├── Home.vue       # 홈 페이지
│   │   ├── vues/
│   │   │   ├── mainPage.vue    # 메인 페이지
│   │   │   └── pickMem.vue     # 사진 편집 메인 페이지
│   │   └── steps/         # 단계별 컴포넌트
│   │       ├── stepOne.vue     # 1단계: 이미지 업로드
│   │       ├── stepTwo.vue     # 2단계: 프레임 선택
│   │       ├── stepThree.vue   # 3단계: 이미지 배치
│   │       ├── stepFour.vue    # 4단계: 스티커/패턴 추가
│   │       ├── stepFive.vue    # 5단계: 최종 확인
│   │       ├── stepResult.vue  # 결과 페이지
│   │       ├── frames/         # 프레임 관련 컴포넌트
│   │       └── vues/           # 단계별 서브 컴포넌트
│   ├── router/            # 라우터 설정
│   │   └── index.js
│   ├── store/             # Vuex 스토어
│   │   └── index.js
│   ├── plugins/           # Vue 플러그인
│   │   ├── index.js
│   │   └── utils.js
│   ├── utils/             # 유틸리티 함수
│   │   └── meta.js        # 메타 태그 설정
│   ├── App.vue            # 루트 컴포넌트
│   └── main.js            # 엔트리 포인트
├── package.json
├── vue.config.js          # Vue CLI 설정
└── babel.config.js        # Babel 설정
```

## 🎯 주요 기능

### 1. 이미지 업로드 및 편집
- 사용자 이미지 업로드
- Fabric.js를 활용한 이미지 조작 (크기 조절, 이동, 회전 등)

### 2. 프레임 선택
- 다양한 폴라로이드 스타일 프레임 제공
- 그리드 레이아웃 선택 (행/열 설정)

### 3. 이미지 배치
- 드래그 앤 드롭으로 이미지 배치
- 여러 이미지를 프레임에 배치

### 4. 스티커 및 패턴 추가
- 다양한 스티커 제공 (꽃잎, 귀여운 일러스트 등)
- 패턴 배경 적용
- 스티커 위치 조정 및 삭제

### 5. 결과물 저장
- html2canvas를 사용하여 편집된 결과를 이미지로 저장
- PNG 형식으로 다운로드

## 🛠️ 설치 및 실행

### 필수 요구사항
- Node.js (v14 이상 권장)
- npm 또는 yarn

### 설치

```bash
# 의존성 패키지 설치
npm install
```

### 개발 서버 실행

```bash
# 개발 모드로 실행 (hot-reload 지원)
npm run serve
```

개발 서버는 기본적으로 `http://localhost:8080`에서 실행됩니다.

### 프로덕션 빌드

```bash
# 프로덕션용 빌드
npm run build
```

빌드된 파일은 `../docs` 디렉토리에 생성됩니다.

## 📦 주요 의존성

### 프로덕션 의존성
- `vue`: Vue.js 프레임워크
- `vue-router`: 라우팅
- `vuex`: 상태 관리
- `fabric`: 캔버스 기반 이미지 편집
- `html2canvas`: 캔버스를 이미지로 변환
- `vuedraggable`: 드래그 앤 드롭
- `vue-toasted`: 토스트 알림
- `vue-meta`: 메타 태그 관리

### 개발 의존성
- `@vue/cli-service`: Vue CLI 서비스
- `@vue/cli-plugin-babel`: Babel 플러그인
- `@vue/cli-plugin-router`: Vue Router 플러그인
- `@vue/cli-plugin-vuex`: Vuex 플러그인
- `sass`: Sass 컴파일러
- `sass-loader`: Sass 로더
- `vue-template-compiler`: Vue 템플릿 컴파일러

## 🗂️ 상태 관리 (Vuex)

주요 상태:
- `canNext`: 다음 단계로 진행 가능 여부
- `rows`, `columns`: 그리드 레이아웃 설정
- `frameCanvas`: 프레임 캔버스 객체
- `frameImg`: 프레임 이미지
- `frame`: 선택된 프레임 정보
- `images`: 업로드된 이미지들
- `imgCanvas`: 이미지별 캔버스 객체
- `selectTarget`: 선택된 타겟 정보
- `selectList`: 선택된 항목 리스트
- `tmpTarget`: 임시 타겟 정보
- `orderQueue`: 순서 큐

## 🎨 에셋

### 프레임
- 다양한 레이아웃의 폴라로이드 프레임 (1_2.png, 1_3.png, 2_1.png 등)

### 스티커
- 꽃잎 스티커 (flower_leaf_1.png ~ flower_leaf_38.png)
- 귀여운 일러스트 스티커 (cute_handdrawn, cute_natural_doodle 시리즈)

### 패턴
- 9가지 배경 패턴 (pattern_1.png ~ pattern_9.png)

## 🔧 설정

### vue.config.js
- `outputDir`: 빌드 출력 디렉토리 (`../docs`)
- `publicPath`: 프로덕션 환경에서 `./` 사용

### 브라우저 지원
- 최신 2개 버전의 브라우저
- 점유율 1% 이상의 브라우저
- 지원 중단되지 않은 브라우저

## 📝 라이선스

이 프로젝트는 private 프로젝트입니다.

## 👤 작성자

dnfwlxo11

---

**Pick The Moment** - 소중한 순간을 특별하게 만들어보세요! 📸✨
