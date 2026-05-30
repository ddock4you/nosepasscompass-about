# 코코파스의 나침반 (Nosepass Compass)

> 한국어 사용자를 위한 1~9세대 포켓몬 도감·기술·특성·도구·타입 상성 학습 웹앱

Next.js App Router 기반으로 1~9세대 포켓몬 데이터를 정적 JSON 데이터셋으로 재구성하고, 대량 SSG (약 9000 페이지), 클라이언트 검색 인덱스 압축, Cloudflare Workers 비용 최적화를 적용한 모바일 우선 포켓몬 학습 웹앱입니다.

[![Next.js](https://img.shields.io/badge/Next.js-16.2-000?logo=next.js)](https://nextjs.org)
[![React](https://img.shields.io/badge/React-19.2-149ECA?logo=react)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?logo=typescript)](https://www.typescriptlang.org)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-4.1-38BDF8?logo=tailwindcss)](https://tailwindcss.com)
[![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers%20%2B%20OpenNext-F38020?logo=cloudflare)](https://workers.cloudflare.com)

---

## 링크

- **Live**: <https://nosepasscompass.com>

### 추천 체험 흐름

1. `/search`에서 `리자몽` 검색
2. `/dex/6/gen/9` 상세 페이지에서 세대별 정보 확인
3. `다른 모습`에서 메가 X/Y 폼 전환
4. `/training`에서 타입 상성 퀴즈 체험
5. `/type-chart`에서 18종 타입 상성표 확인

---

## 프로젝트 핵심 요약

| 구분            | 내용                                                                                                    |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| 목적            | 한국어 사용자가 1~9세대 포켓몬 정보를 모바일 환경에서 빠르게 검색·학습할 수 있는 통합 웹앱 구축         |
| 핵심 설계       | 런타임 API 의존 제거, 빌드 타임 데이터셋 생성, 대량 SSG, 정적 자산 기반 데이터 제공                     |
| 주요 최적화     | 검색 인덱스 압축, Cloudflare Workers 호출 비용 절감, 이미지·폰트 최적화                                 |
| 프론트엔드 구조 | Next.js App Router, 서버 컴포넌트 우선, Feature-Sliced Design (FSD Archtecture), Zustand 기반 상태 분리 |
| 운영 환경       | Cloudflare Workers + OpenNext, GitHub Actions, Wrangler 배포                                            |

---

## 주요 성과

- **런타임 API 의존 제거**  
  PokéAPI를 런타임에서 직접 호출하지 않고, 빌드 타임 데이터 소스로만 사용하도록 전환했습니다. 서비스 런타임은 자체 정적 JSON 데이터셋과 사전 계산된 파생 데이터를 기반으로 동작합니다.

- **대량 SSG 적용**  
  포켓몬 상세 페이지를 `/dex/[id]/gen/[gen]` 구조로 설계하고, 포켓몬 × 가용 세대 조합을 정적 생성했습니다. 세대별 기술 학습, 획득 경로, 소지 도구 차이를 URL 단위로 분리했습니다.

- **검색 인덱스 약 84% 축소**  
  포켓몬·기술·특성·도구 검색 데이터를 JSON → MessagePack → Brotli 파이프라인으로 압축해 약 704KB에서 약 112KB로 줄였습니다. 검색 인덱스는 정적 import하지 않고 필요한 시점에 ASSETS에서 로드합니다.

- **Cloudflare Workers 호출 비용 최적화**  
  Next.js 기본 prefetch와 ISR이 Worker invocation에 미치는 영향을 분석하고, `prefetch={false}`, `incrementalCache: "dummy"`, per-id JSON 분산 전략을 적용해 호출 비용을 줄였습니다.

- **상태 관리 구조 개선**  
  Context API 기반 전역 상태의 re-render와 hydration mismatch 가능성을 줄이기 위해 Zustand store를 도메인별로 분리했습니다. Preferences, Quiz, Dex Filter, Moves Filter 상태를 각각 라우트 생명주기에 맞춰 관리했습니다.

- **모바일 우선 UI 설계**  
  320px 환경에서도 표와 카드 UI가 깨지지 않도록 컬럼 전략과 터치 타깃을 재설계했습니다. 모든 페이지는 `max-w-3xl` 기준으로 중앙 정렬하고, 주요 인터랙션은 44×44px 이상의 터치 영역을 확보했습니다.

---

## 프로젝트 배경

기존 포켓몬 도감 사이트들은 정보량은 많지만, 한국어 사용자가 모바일 환경에서 세대별 데이터를 빠르게 비교하고 학습하기에는 몇 가지 한계가 있었습니다.

- 런타임 PokéAPI 의존으로 인한 한국어 데이터 일관성, 검색 품질, SEO, 속도 한계
- 특정 세대 중심 정보 구성으로 인한 1~9세대 통합 검색·비교의 어려움
- 데스크톱 중심 UI로 인한 모바일 표 깨짐, 좁은 터치 영역, 낮은 탐색성

이 프로젝트는 위 문제를 해결하기 위해 외부 API를 런타임에 직접 호출하지 않고, 빌드 타임에 수집·검증·보강한 정적 데이터셋을 기반으로 동작하도록 설계했습니다.

---

## 주요 기능

### 통합 검색

포켓몬, 기술, 특성, 도구를 한 번에 검색할 수 있습니다. 메가진화, 거다이맥스, 리전폼 같은 별칭과 영문·일문 검색어를 함께 매칭합니다.

### 포켓몬 도감

세대, 타입, 정렬 기준을 조합해 포켓몬을 탐색할 수 있습니다. 무한 스크롤과 수동 더보기 fallback을 제공하며, 가용 세대와 폼 정보를 카드 단위로 표시합니다.

### 포켓몬 상세

포켓몬 상세 페이지는 개요, 기술, 진화·입수, 육성·기타 4개 탭으로 구성됩니다. 세대별로 기술 학습, 획득 경로, 소지 도구가 달라지기 때문에 `/dex/[id]/gen/[gen]`을 canonical URL로 사용했습니다.

### 세대 전환

세대 정보는 URL path segment를 SSOT로 사용합니다. 사용자의 선호 세대는 쿠키와 localStorage에 저장하고, 유효하지 않은 폼·세대 조합은 canonical URL로 리다이렉트합니다.

### 배틀 트레이닝 퀴즈

타입 상성, 포켓몬 속성 퀴즈를 제공합니다. 퀴즈 진행 중에는 뒤로가기, 검색, 메뉴 이동 시 confirm 다이얼로그를 띄워 사용자의 진행 상태 손실을 방지합니다.

### 타입 상성표

18종 타입 상성을 카드 그리드로 제공합니다. 1배 보통 상성은 숨기고, 강함·약함·무효 관계만 강조해 모바일에서도 빠르게 확인할 수 있도록 설계했습니다.

---

## 기술 스택

### Frontend

- **Next.js 16.2**: App Router, 서버 컴포넌트 우선 구조, 대량 SSG
- **React 19.2**: 클라이언트 인터랙션, Suspense 기반 UI 분리
- **TypeScript 5.9**: strict 기반 도메인 모델링
- **Zustand 5**: Preferences, Quiz, Dex Filter, Moves Filter 상태 분리
- **TanStack Query 5**: 검색 인덱스와 기술 학습 데이터 캐시
- **Tailwind CSS 4 + shadcn/ui**: 모바일 우선 UI와 vendor wrapper 패턴
- **Radix UI / Sonner**: Dialog, Tabs, Popover, Toast 등 접근성 기반 UI 구성

### Infra / Deploy

- **Cloudflare Workers + OpenNext**: Next.js 앱을 Workers 환경에 배포
- **Cloudflare Static Assets Binding**: 정적 자산과 per-id JSON을 Worker 호출 없이 제공
- **GitHub Actions + Wrangler**: master push 기반 자동 배포
- **Custom Domain**: <https://nosepasscompass.com>

### Build / Tooling

- **pnpm 11**: 패키지 매니저 고정
- **Vitest**: shared lib, entity, 데이터 변환 로직 테스트
- **ESLint 9**: Next.js, React Hooks, React Refresh 규칙 적용
- **Sharp**: 포켓몬 이미지 WebP 3-variant 변환
- **MessagePack + Brotli**: 검색 인덱스 압축
- **Cheerio**: 외부 데이터 크롤링 파서

---

## 아키텍처 하이라이트

### 1. Cloudflare Workers 호출 비용 최적화

**문제**  
Next.js 기본 `<Link>`는 viewport 진입 시 RSC payload를 자동 prefetch합니다. 카드가 많은 도감 화면에서 100개 이상의 링크가 동시에 RSC fetch를 발생시키면 Cloudflare Workers invocation이 급증할 수 있습니다.

**접근**  
모든 `<Link>`에 `prefetch={false}`를 적용하고, grep 기반 검사로 규칙을 강제했습니다. 또한 `open-next.config.ts`에서 `incrementalCache: "dummy"`를 설정해 ISR을 비활성화하고, 큰 단일 JSON import 대신 `public/data/pokemon/<id>.json` 단위의 per-id JSON을 ASSETS에서 가져오도록 분리했습니다.

**결과**  
HTML 라우트 중심으로 Worker 호출을 제한하고, 정적 자산·이미지·폰트·per-id JSON은 Cloudflare Static Assets 경로로 제공했습니다. 큰 JSON 정적 import를 제거해 cold-start CPU 부담도 줄였습니다.

---

### 2. 검색 인덱스 3단 압축 파이프라인

**문제**  
포켓몬·기술·특성·도구 검색 데이터를 원본 JSON 그대로 클라이언트 번들에 포함하면 초기 로딩 비용이 커지고, 검색을 사용하지 않는 사용자도 불필요한 비용을 지불하게 됩니다.

**접근**  
`build-search-index.ts`에서 도메인별 슬림 JSON을 생성한 뒤 MessagePack으로 바이너리 인코딩하고, Brotli quality 11로 압축했습니다. 생성된 파일은 content-hash 파일명과 manifest로 관리해 빌드마다 자동 invalidation되도록 구성했습니다.

**결과**  
검색 인덱스 크기를 약 704KB에서 약 112KB로 줄였습니다. 검색 데이터는 클라이언트 번들에 정적 import하지 않고, 검색 기능 사용 시 ASSETS에서 로드합니다.

---

### 3. Context API에서 Zustand로 상태 관리 전환

**문제**  
Preferences, Quiz, Filter 상태를 Context API로 관리하면 provider 범위가 커지고 값 객체 변경으로 불필요한 re-render가 발생할 수 있습니다. 또한 localStorage lazy initializer는 React 19 환경에서 hydration mismatch를 만들 가능성이 있습니다.

**접근**  
클라이언트 UI 상태를 Zustand store 4개로 분리했습니다.

- `preferencesStore`: 테마, 선호 세대, 타입표, 기술 상세 VG
- `quizStore`: `/training` 퀴즈 진행 상태
- `dexFilterStore`: `/dex` 검색어, 세대·타입·정렬·페이지 필터
- `movesFilterStore`: `/moves` 타입·분류 필터

selector hook과 안정적인 actions 객체 패턴을 사용하고, storage sync는 필요한 setter에서 직접 처리했습니다.

**결과**  
상태 구독 범위를 필드 단위로 줄였고, 다크모드·선호 세대·기술 VG는 FOUC 없이 반영되도록 구성했습니다. 도감, 기술, 퀴즈 상태는 라우트 생명주기에 맞춰 명시적으로 reset됩니다.

---

### 4. 다출처 데이터 파이프라인과 한국어 보충 워크플로우

**문제**  
PokéAPI 단일 출처만으로는 한국어 기술 설명, 특성, 도구, 장소명, 일부 세대의 획득 경로 데이터가 부족했습니다.

**접근**  
출처별 역할을 나누고, 사람이 검토할 수 있는 staging 파일 패턴을 도입했습니다.

- **PokéAPI**: 기본 골격, 스탯, 타입, 기술 학습, 진화 체인
- **PokemonDB**: 스탯·타입 재검증, 기술 한국어명 보충
- **Serebii**: 일부 세대의 야생 인카운터 데이터
- **Bulbapedia**: 장소명 번역 보조
- **pokemon.fandom.com/ko**: 기술 한국어 flavor text

한국어 보충 데이터는 `_overrides.ko.json` 같은 staging 파일에 모으고, 적용 스크립트는 빈 필드만 덮어쓰도록 설계했습니다.

**결과**  
한국어 데이터 일관성을 유지하면서도 출처 추가·교체가 가능한 데이터 인벤토리를 구축했습니다.

---

### 5. 포켓몬 × 세대 조합 대량 SSG

**문제**  
포켓몬 상세 정보는 세대마다 기술 학습, 획득 경로, 소지 도구가 달라집니다. 단일 상세 페이지로는 SEO와 세대 비교 경험을 모두 만족시키기 어렵습니다.

**접근**  
canonical URL을 `/dex/[id]/gen/[gen]`으로 정의하고, 실제 가용 세대 배열 기반으로 존재하는 조합만 `generateStaticParams`에서 prerender했습니다. `/dex/[id]` 진입 시에는 쿠키와 기본 세대를 기준으로 서버 사이드 리다이렉트합니다.

**결과**  
포켓몬 상세 페이지를 species × available generations 조합으로 정적 생성했습니다. 동적 라우트는 선호 세대 리다이렉트를 처리하는 `/dex/[id]` 하나로 제한했습니다.

---

### 6. 모바일 우선 UI와 정적 자산 최적화

**문제**  
도감류 서비스는 표, 스탯, 기술 리스트처럼 정보 밀도가 높은 UI가 많아 320px 모바일 환경에서 쉽게 깨집니다. 한글 폰트와 포켓몬 이미지도 초기 로딩 성능에 영향을 줍니다.

**접근**  
모든 페이지를 `max-w-3xl` 기준으로 설계하고, 터치 타깃은 44×44px 이상을 유지했습니다. 기술표는 320px에서도 한 줄로 보이도록 PP·메타 열을 합치는 반응형 컬럼 전략을 적용했습니다. Pretendard Variable은 woff2-dynamic-subset을 자체 호스팅하고, 포켓몬 이미지는 WebP 3-variant로 변환했습니다.

**결과**  
모바일 환경에서 정보 밀도와 가독성을 함께 확보했고, 이미지·폰트 로딩 비용을 줄였습니다.

---

## 데이터셋 규모

| 영역      | 규모                                               |
| --------- | -------------------------------------------------- |
| 포켓몬    | 1,399개 JSON: 1~1025번 + 폼·메가·거다이맥스·리전폼 |
| 기술      | 919건                                              |
| 특성      | 전 세대 통합                                       |
| 도구      | 전 세대 통합                                       |
| 장소      | Gen 1~9 + 한국어·일본어 번역 보충                  |
| 진화 체인 | form-aware multi-leaf 정규화                       |
| 성격      | 25건                                               |

빌드 후 산출물 합계는 약 77MB raw이며, 클라이언트에는 압축 검색 인덱스와 per-id JSON 단위로 필요한 데이터만 전달합니다.

---

## 데이터 파이프라인

```text
PokéAPI (빌드 타임 데이터 소스)
        │
        ▼
(수집) populate-from-pokeapi.ts
        │
        ├── PokemonDB
        ├── Serebii
        ├── Bulbapedia
        └── pokemon.fandom.com/ko
        │
        ▼
(검증) _overrides.ko.json / locations 한국어 staging
        │
        ▼
(보강) fill-moves-ko.ts / apply-locations-ko.ts
        │
        ▼
(적용) src/shared/data/**/*.json
        │
        ▼ pnpm prebuild
        │
        ├── public/data/**/*.json
        ├── src/shared/data/search/*.msgpack.br
        ├── src/shared/data/quiz/*
        └── derived lookup JSON
```

데이터 파이프라인은 수집, 검증, 보강, 적용 단계를 분리했습니다. 한국어 보충 데이터는 staging 파일에서 사람이 검토한 뒤 적용하며, 적용 스크립트는 빈 필드만 채우는 불변 contract를 따릅니다.

---

## 프론트엔드 구조

경량 Feature-Sliced Design을 적용해 라우트, 기능, 도메인 모델, 공통 인프라를 분리했습니다.

```text
src/
├── app/         # Next.js App Router 라우트 엔트리
├── features/    # 사용자 인터랙션 단위
├── entities/    # 도메인 데이터 모델
└── shared/      # 공통 UI, lib, config, data, api
```

### 의존성 규칙

- `app` → `features` / `entities` / `shared`
- `features` → `entities` / `shared`
- `entities` → `shared`
- feature 간 import 금지
- entity 간 import 금지
- 크로스슬라이스 공유 모듈은 `shared/`로 이동

이 구조를 통해 화면 단위 feature를 수정하거나 제거할 때 다른 feature에 미치는 영향을 줄였습니다.

---

## 배포와 운영

GitHub Actions 워크플로우가 `master` push 시 자동 실행됩니다.

```text
1. checkout
2. pnpm install --frozen-lockfile
3. 캐시 복원
   - artwork-webp
   - .next/cache
   - public/data
4. pnpm build:worker
5. wrangler deploy
```

### 렌더링 전략

- **SSG**: 포켓몬 상세, 폼 상세, 기술 상세, 특성 상세, 도구 상세
- **Static**: 홈, 도감, 기술, 특성, 도구, 검색, 트레이닝, 타입 상성표, 정책 페이지
- **Dynamic**: `/dex/[id]` 단일 리다이렉트 라우트

---

## 테스트와 품질 관리

- **Vitest**: entity, shared lib, 데이터 변환 유틸 테스트
- **Testing Library + jsdom**: 컴포넌트 단위 테스트 환경
- **ESLint 9**: Next.js, React Hooks, React Refresh 규칙 적용
- **TypeScript strict**: 전 영역 strict 통과
- **케이스 검증 스크립트**: Windows와 Linux 파일 시스템 차이로 인한 import casing 오류 방지
- **design.md lint**: 디자인 토큰 문서 lint

---

## 보안과 SEO

### 보안

- Content-Security-Policy 적용
- Strict-Transport-Security 적용
- X-Frame-Options, X-Content-Type-Options, Referrer-Policy 설정
- Cross-Origin 관련 보안 헤더 설정
- Cloudflare WAF Custom Rule로 `/data/*` 직접 크롤링 차단

### SEO

- 라우트별 `generateMetadata`
- 공통 메타데이터 헬퍼 `buildPageMetadata()`
- JSON-LD 자동 주입
- OpenGraph, Twitter Card, canonical 설정
- 세대별 description, keywords 구성

---

## 로컬 개발

### 전제 조건

- Node.js 24+
- pnpm 11+

### 실행 방법

```bash
pnpm install
pnpm dev
pnpm build
pnpm test
pnpm lint
pnpm preview
pnpm deploy
```

### prebuild 주요 작업

`pnpm build` 실행 시 다음 작업을 자동으로 수행합니다.

- import casing 검사
- Pretendard font 복사
- JSON 데이터셋을 `public/data/`로 분산 복사
- 파생 lookup JSON 사전 계산
- 포켓몬 이미지 WebP 변환
- 퀴즈 풀 생성
- 검색 인덱스 MessagePack + Brotli 압축
- ads.txt 생성

---

## 알려진 한계와 개선 예정

- Cloudflare CDN 서버 캐시는 현재 어댑터 동작 특성상 제한적으로 적용됩니다. 브라우저 캐시를 우선 사용하고, 향후 OpenNext 어댑터 변경에 맞춰 재검토할 예정입니다.
- Pretendard dynamic subset은 드물게 자주 쓰이지 않는 한글 chunk가 늦게 로드될 수 있습니다.
- 일부 일본판, 외전, DLC 버전그룹 데이터는 의도적으로 제외했습니다.
- 일부 기술·도구의 한국어 데이터 보충을 계속 진행 중입니다.
