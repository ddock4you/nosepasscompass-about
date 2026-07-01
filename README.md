# 코코파스의 나침반 (Nosepass Compass)

> 대량 정적 데이터와 Cloudflare Workers 비용 최적화를 적용한 포켓몬 정보 탐색 웹앱

Next.js App Router 기반으로 1~9세대 포켓몬 데이터를 정적 JSON 데이터셋으로 재구성하고, **약 9,000개 정적 페이지 생성**, **검색 인덱스 약 84% 축소**, **Cloudflare Workers invocation 최적화**를 적용한 모바일 우선 정보 탐색 웹앱입니다.

[![Next.js](https://img.shields.io/badge/Next.js-16.2-000?logo=next.js)](https://nextjs.org)
[![React](https://img.shields.io/badge/React-19.2-149ECA?logo=react)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?logo=typescript)](https://www.typescriptlang.org)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-4.1-38BDF8?logo=tailwindcss)](https://tailwindcss.com)
[![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers%20%2B%20OpenNext-F38020?logo=cloudflare)](https://workers.cloudflare.com)

---

## 링크

- **라이브 서비스**: [https://nosepasscompass.com](https://nosepasscompass.com)

---

## 프로젝트 개요

기존 포켓몬 정보 사이트들은 정보량은 많지만, 한국어 사용자가 모바일 환경에서 세대별 데이터를 빠르게 비교하고 학습하기에는 몇 가지 한계가 있었습니다. 런타임 API 의존, 한국어 데이터 일관성, 모바일 표 UI, 검색 성능, 배포 비용이 대표적인 문제였습니다.

코코파스의 나침반은 포켓몬이라는 친숙한 소재를 바탕으로, **대량 정적 데이터 기반 정보 탐색 서비스**를 어떻게 설계하고 배포할 수 있는지 검증한 프로젝트입니다. 서비스 런타임에서는 외부 API를 직접 호출하지 않고, 빌드 타임에 수집·검증·보강한 정적 데이터셋을 기반으로 동작합니다.

---

## 핵심 요약

- **런타임 API 의존 제거**: PokéAPI를 빌드 타임 데이터 소스로만 사용하고, 서비스는 정적 데이터셋 기반으로 동작
- **대량 SSG 적용**: 포켓몬 × 가용 세대 조합으로 약 9,000개 정적 페이지 생성
- **검색 인덱스 약 84% 축소**: JSON → MessagePack → Brotli 파이프라인으로 약 704KB → 약 112KB 압축
- **Cloudflare Workers 비용 최적화**: Link prefetch 제어, ISR 비활성화, per-id JSON 정적 자산 분리
- **상태 관리 구조 개선**: Context API 전역 상태를 Zustand 도메인 store로 분리
- **모바일 우선 UI**: 320px 환경에서도 표와 카드 UI가 깨지지 않도록 정보 밀도와 터치 영역 조정

---

## 주요 기능

- 포켓몬, 기술, 특성, 도구 통합 검색
- 1~9세대 포켓몬 도감과 세대별 상세 페이지
- 메가진화, 거다이맥스, 리전폼 등 폼 전환
- 세대별 기술 학습, 획득 경로, 소지 도구 정보 제공
- 타입 상성표와 배틀 트레이닝 퀴즈
- 사용자 선호 세대, 테마, 필터 상태 관리
- metadata, canonical, Open Graph, JSON-LD 기반 SEO 구성

---

## 핵심 설계 판단

### 1. 런타임 API 의존을 제거한 정적 데이터 구조

**문제**  
런타임에서 PokéAPI를 직접 호출하면 한국어 데이터 일관성, 응답 속도, 검색 품질, SEO, 장애 대응 측면에서 한계가 있습니다. 또한 세대별 정보와 폼 데이터를 화면에서 매번 조합하면 클라이언트 부담이 커질 수 있습니다.

**접근**  
PokéAPI는 빌드 타임 데이터 소스로만 사용하고, 서비스 런타임에서는 자체 정적 JSON 데이터셋과 사전 계산된 파생 데이터를 사용하도록 전환했습니다. 부족한 한국어 데이터는 별도 보강 파일과 적용 스크립트로 관리했습니다.

**결과**  
서비스 런타임에서 외부 API 의존을 제거하고, 검색·상세·SEO에 필요한 데이터를 정적 자산으로 제공할 수 있게 했습니다. 배포 후에는 정적 데이터 기반으로 안정적으로 동작하는 구조를 만들었습니다.

---

### 2. 세대별 정보를 URL로 분리한 대량 정적 생성

**문제**  
포켓몬 상세 정보는 세대마다 기술 학습, 획득 경로, 소지 도구가 달라집니다. 단일 상세 페이지로는 세대별 정보 탐색과 SEO를 모두 만족시키기 어렵습니다.

**접근**  
canonical URL을 `/dex/[id]/gen/[gen]`으로 정의하고, 실제 가용 세대 조합만 `generateStaticParams`에서 정적 생성했습니다. `/dex/[id]` 진입 시에는 쿠키와 기본 세대를 기준으로 적절한 세대 페이지로 리다이렉트합니다.

**결과**  
약 9,000개의 세대별 상세 페이지를 정적으로 생성했습니다. 세대별 데이터를 URL 단위로 분리해 검색 엔진과 사용자 모두에게 명확한 정보 구조를 제공했습니다.

---

### 3. 검색 인덱스 압축과 Workers 호출 비용 최적화

**문제**  
검색 데이터를 원본 JSON 그대로 클라이언트 번들에 포함하면 초기 로딩 비용이 커집니다. 또한 Next.js 기본 Link prefetch와 ISR은 카드가 많은 도감 화면에서 Cloudflare Workers invocation을 불필요하게 증가시킬 수 있습니다.

**접근**  
검색 인덱스는 도메인별 슬림 JSON을 생성한 뒤 MessagePack으로 인코딩하고 Brotli로 압축했습니다. 검색 기능 사용 시에만 ASSETS에서 로드하도록 분리했습니다. 또한 `<Link prefetch={false}>`, `incrementalCache: "dummy"`, per-id JSON 정적 자산 분산 전략을 적용했습니다.

**결과**  
검색 인덱스를 약 704KB에서 약 112KB로 줄였고, Worker는 HTML 라우트 중심으로만 호출되도록 제한했습니다. 정적 자산, 이미지, 폰트, per-id JSON은 Cloudflare Static Assets 경로로 제공해 배포 비용과 cold-start 부담을 줄였습니다.

---

## 기술 스택

| Area           | Stack                                                            |
| -------------- | ---------------------------------------------------------------- |
| Framework      | Next.js 16.2, React 19.2, TypeScript 5.9                         |
| Styling        | Tailwind CSS 4, shadcn/ui, Radix UI, Sonner                      |
| State & Data   | Zustand, TanStack Query, static JSON dataset                     |
| Build Pipeline | Sharp, MessagePack, Brotli, Cheerio                              |
| Infra          | Cloudflare Workers, OpenNext, Cloudflare Static Assets, Wrangler |
| Quality        | Vitest, ESLint, GitHub Actions                                   |

---

## 추천 확인 흐름

1. `/search`에서 `리자몽` 검색
2. `/dex/6/gen/9` 상세 페이지에서 세대별 정보 확인
3. `다른 모습`에서 메가 X/Y 폼 전환
4. `/training`에서 타입 상성 퀴즈 체험
5. `/type-chart`에서 18종 타입 상성표 확인
6. 모바일 화면 폭에서 도감 카드, 기술표, 타입표의 반응형 UI 확인

---

## 현재 상태

- 라이브 서비스 운영 중
- 정적 데이터셋 기반 검색·도감·상세·퀴즈 주요 흐름 구현 완료
- 향후 데이터 보강, 검색 품질 개선, 모바일 UI 세부 개선을 점진적으로 진행할 수 있는 구조

---

## 로컬 실행 방법

```bash
pnpm install
pnpm dev
pnpm build
pnpm test
pnpm lint
pnpm preview
pnpm deploy
```

`pnpm build` 실행 시 데이터 전처리, 검색 인덱스 생성, 정적 페이지 빌드, Workers 배포 산출물 생성 흐름이 함께 실행됩니다.
