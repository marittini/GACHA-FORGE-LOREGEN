# v1.2.0 검증

## 통과
- node --check 문법 검사.
- jsdom 실제 이벤트 + 모의 API:
  - 이전 15개 초안에서 최근 10개로 제한, 삭제.
  - 플로팅/도크 버튼 없음, 확장 이름 변경.
  - 생성 지침 편집 및 실제 모델 요청 포함.
  - 프리셋 저장·선택 불러오기·기본값 복원·삭제.
  - 등록 후 서버 목록 읽기 확인.
  - QueryClient bridge의 항목 캐시 반영/로어북 조회 무효화.
  - 성공 후 입력 초기화 및 지침·프리셋 유지.
  - 등록 실패 시 입력 보존.
- Chromium 133 + Playwright + 모의 API:
  - 로어북 편집 아이콘 실행 및 대상 선택.
  - 실제 지침 입력·프리셋 등록·생성 요청.
  - 등록 확인 뒤 빈 입력 화면으로 전환.
  - 자동 캐시 갱신 불가 시 명시적 화면 새로고침 버튼.
  - 초안 삭제, 모바일 390×844 가로 넘침 없음 및 주요 버튼 화면 내 위치.
  - 라이트·다크 화면 렌더링 확인, 브라우저 오류 없음.

## 원인 및 소스
2026-10-12 GitHub main의 packages/client/src/hooks/use-lorebooks.ts:
- useLorebooks staleTime = 5 * 60_000.
- Engine 기본 등록 hook은 onSuccess에서 query invalidation을 실행하지만 기존 확장의 직접 fetch는 이를 실행하지 않음.
- entries query key: ['lorebooks','entries',lorebookId].

## 범위와 한계
실제 사용자 설치 서버·모델 및 실제 Engine React provider와의 통합 검사는 수행하지 않았습니다. 캐시 bridge는 알려진 provider 구조를 모의 객체로 검사했습니다. 공개 API가 아닌 선택적 탐지 경로이므로 Engine 변경 시 실패할 수 있으며, 실패할 때는 등록 데이터를 보존하고 명시적인 화면 새로고침 버튼으로 안내합니다.
시간 표시는 서버 저장 확인까지의 시간만 측정합니다. 실제 사용자 환경의 지연값을 대신 측정한 것은 아닙니다.

# v1.3.0 추가 검증

- node --check 통과.
- jsdom + 모의 API: 현재 프로필 비우기 후 보관함 유지, 웹검색 기본 꺼짐, 미지원 연결에서 호출 중단, Gemini googleSearch 옵션 전달, OpenRouter Exa 및 도메인 제한 전달, 모델 반환 자료 먼저 검토하는 흐름, 안전한 링크·도메인·출처 연결된 사실 필터, 자료의 생성 요청 포함, 검색어 변경 시 다시 검색, 빈 검색 자료에서 생성 중단, 꺼짐 상태에서 자료 제외 및 검색 옵션/online 접미사 제거.
- v1.2 초안 삭제/10개·프리셋·등록 초기화/실패 보존 회귀 검사 통과.
- Chromium 133 + 모의 API: 검색 자료 화면 → 검토한 자료로 생성 → 등록 초기화 → 수동 작성 → 비우기, 모바일 390×844 가로 넘침/주요 버튼 위치 확인, 브라우저 런타임 오류 없음.
- 검색 실호출 및 실제 계정 비용 검증은 수행하지 않았음. 검색 도구 실행·출처 메타데이터는 현재 Engine raw API가 전달하지 않으므로 독립 검증하지 못함. Google groundingMetadata나 OpenRouter annotations를 직접 받았다고 주장하지 않음.
- 소스 대조: routes/generate/raw-route.ts의 customParameters 지원 및 텍스트-only 반환, google.provider.ts의 요청 본문 customParameters 병합, base-provider.ts의 customParameter 병합, OpenRouter 공식 web plugin 문서 및 Google 공식 검색 grounding 문서.

# v1.4.0 추가 검증

- node --check 통과.
- 현재 저장된 v1.3.0 원본과 이후 사용자 업로드 Gacha-Forge-lorebookgenerate(2).zip의 extension.js가 동일함을 바이트 비교로 확인하고 이를 기준으로 수정. 설치 manifest는 사용자 export의 marinara.personal-extension/client/full_page_access 형식을 유지.
- jsdom 실제 UI 이벤트 + 모의 API 검사 통과:
  - 검색 수집 지침 입력이 검색용 모델 요청에 포함됨.
  - 기존 작성 지침 및 작성 프리셋을 유지하며 검색 프리셋을 독립 저장/불러오기/삭제/기본값 복원.
  - 7개 프로필 항목에 대응하는 sections/missing JSON 계약 및 외형 우선 검색 목표 전달.
  - 머리·눈·흉터·복장·귀걸이 자료와 미확인 키 정보를 항목별로 표시.
  - 반환된 출처 URL과 연결되지 않은 사실/위험 URL 제외.
  - 외형의 모든 모의 수집 특징이 후속 프로필 생성 요청에 그대로 들어감.
  - 구체적인 외형을 모호한 요약으로 대체하지 않도록 생성 지시 포함.
  - 검색 지침 수정/프리셋 불러오기/복원 후 검색 자료 무효화.
  - v1.3 요약 결과 자동 재사용 방지.
  - 외형 자료 누락 경고, 일반 요약만 반환하면 중단하고 현재 프로필 유지.
  - 비우기 후 검색 지침 유지.

실제 검색 서비스·실제 생성 모델을 호출하지 않았습니다. 이 턴에서는 jsdom 검사를 수행했으며 실제 Chromium 렌더링 검사는 수행하지 않았습니다. 과거 v1.3 브라우저 검사 기록은 당시 버전의 검증 기록입니다. 검색 실행/출처 진위는 Engine raw API의 메타데이터 제한으로 여전히 독립 검증하지 못합니다.
