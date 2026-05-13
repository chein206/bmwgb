# BMW GB 차량 위치 보드 — 프로젝트 문서

> 한독모터스 강북전시장 내부용 차량 위치 관리 웹앱  
> 작성일: 2026-05-13

---

## 1. 프로젝트 개요

### 배경
- 전시장 내 화이트보드에 구역별로 차량 자석(마커)을 붙여 위치를 관리하던 아날로그 방식
- 직원들이 데스크에 직접 와야만 현황 파악 가능
- **목표**: 어디서든 스마트폰/PC로 실시간 차량 위치 확인 및 수정 가능한 온라인 보드

### 핵심 요구사항
- 접속 즉시 전체 현황 조회 (로그인 불필요)
- 마커 이동은 버튼 토글로 잠금/해제
- 차량 추가·삭제·구역 추가는 관리자 비번 필요
- 모든 직원 기기에서 실시간 동기화
- PC + 모바일 모두 지원

---

## 2. 기술 스택

| 항목 | 선택 | 이유 |
|------|------|------|
| 프론트엔드 | HTML/CSS/JS (Single File) | 별도 빌드 없이 Hugging Face 업로드만으로 배포 |
| 데이터베이스 | Firebase Realtime Database | 실시간 동기화에 최적화, 무료 플랜 충분 |
| 호스팅 | Hugging Face Spaces | 기존 도구들과 동일 환경 |
| 폰트 | Bebas Neue + Noto Sans KR | BMW 감성 + 한글 지원 |

### Firebase 설정
```
Project ID   : bmw-gb-marker
Database URL : https://bmw-gb-marker-default-rtdb.asia-southeast1.firebasedatabase.app
Region       : asia-southeast1 (Singapore)
플랜         : Spark (무료)
```

---

## 3. 권한 구조

```
일반 접속
  └─ 전체 현황 조회 (읽기 전용)

🔓 이동 모드 (버튼 토글)
  └─ 마커 드래그 이동 가능
  └─ 비밀번호 없음, 버튼 한 번으로 on/off

👑 관리자 모드 (비번: 0123)
  └─ 차량(마커) 추가 / 수정 / 삭제
  └─ 구역 추가
  └─ 구역 순서 변경 (드래그)
  └─ 구역 초기화 (기본 구역 복구)
  └─ 관리자 비밀번호 변경
```

---

## 4. 구역(Zone) 구성

### 기본 구역 (9개)
| 순서 | 구역명 | 특이사항 |
|------|--------|----------|
| 1 | 프라자 | 슬롯 1~8번 고정 레이아웃 |
| 2 | 3F | 일반 구역 |
| 3 | 4F | 일반 구역 |
| 4 | 전시 | 일반 구역 |
| 5 | 지점장님 | 일반 구역 |
| 6 | 능이마을 | 일반 구역 |
| 7 | AS입고 | 일반 구역 |
| 8 | 자택보관 | 일반 구역 |
| 9 | 대차 | 일반 구역 |

### 프라자 특수 구조
- 1~8번 슬롯이 세로 1열로 고정 배치
- 각 슬롯에 차량 1대 지정 배치
- 빈 슬롯은 "비어있음" 표시
- 마커를 특정 슬롯 번호에 드롭하면 해당 슬롯에 고정

---

## 5. 차량 마커(Marker) 구조

### 마커 표시 정보 (4줄)
```
1줄: [🚗 신차] or [🏎️ 시승차]  ← 차량 구분 뱃지
2줄: 차종명  외장코드 / 내장코드  ← 색상명 없이 코드만
3줄: 담당자 · 차량번호/차대번호
4줄: 📌 메모
```

### 마커 색상 구분
- **세단**: 파란색 테두리 + 파란 점 (●)
- **SUV**: 초록색 테두리 + 초록 점 (●)
- **시승차**: 빨간색 왼쪽 보더 강조
- **신차**: 파란색 왼쪽 보더 강조

### 마커 정렬 규칙
- 프라자 제외 모든 구역: **시승차가 항상 위에** 표시
- 프라자: 슬롯 번호 순서 고정

### Firebase 마커 데이터 구조
```json
{
  "model": "X5 xDrive40i M Sport 5st",
  "series": "X5",
  "type": "suv",
  "cartype": "new",
  "extColor": "475",
  "intColor": "VASW",
  "plate": "12가 3456 / VIN번호",
  "owner": "김철수",
  "zoneId": "-Nxxx...",
  "slot": 3,
  "memo": "A직원 대차",
  "createdAt": 1700000000000,
  "updatedAt": 1700000000000
}
```

---

## 6. 색상 코드 데이터

> 어프로치북 PDF(BMW_어프로치북_260508.pdf)에서 자동 추출

### 외장 색상 (24종)
| 코드 | 색상명 |
|------|--------|
| 300 | Alpine White |
| 416 | Carbon Black |
| 475 | Black Sapphire |
| A90 | Sophisto Grey |
| A96 | Mineral White |
| C1K | M Marina Bay Blue |
| C1M | Phytonic Blue |
| C31 | M Portimao Blue |
| C34 | San Francisco Red |
| C36 | Individual Dravit Grey |
| C3G | M Toronto Red |
| C4A | Oxide Grey |
| C4F | Artic Race Blue |
| C4G | Isle of Man Green |
| C4P | M Brooklyn Grey |
| C4W | Skyscraper Grey |
| C55 | Sparkling Copper Grey |
| C56 | Thundernight |
| C57 | Aventurine Red |
| C5Y | Cape York Green |
| C67 | Space Silver |
| C68 | Fire Red |
| C7A | Dune Grey |
| C7R | Night Dusk Blue |

### 내장 색상 (33종)
| 코드 | 색상명 |
|------|--------|
| KSFU | Veganza Smoke White |
| KSJ6 | Veganza Calm Beige |
| KSJX | Veganza Espresso Brown |
| KSSW | Veganza Black |
| KUCK | Veganza Oyster |
| KUFU | Veganza Smoke White |
| KUIC | Veganza Castanea |
| KUMY | Veganza Mocha |
| KUSW | Veganza Black |
| LKIA | Merino Silverstone |
| LKKG | Merino Red |
| LKKX | Merino Kyalami Orange |
| LKSW | Merino Black |
| MAEY | Vernasca Ivory White |
| MAH7 | Vernasca Black |
| MAKN | Vernasca Magma Red |
| MAMU | Vernasca Mocha |
| MAPQ | Vernasca Cognac |
| SCHA | Amido |
| SCIC | Castanea |
| SDJL | Atlas Grey |
| VAEW | BMW Individual Merino Ivory White |
| VAHF | BMW Individual Merino Coffee |
| VASW | BMW Individual Merino Black |
| VATQ | BMW Individual Merino Tartufo |
| VCA9 | Silverstone |
| VCDA | Sakhir Orange Black |
| VCJL | Grey |
| VCJM | Merino Copper Brown |
| VCJN | Atlas Grey |
| VCKJ | Merino Amber |
| VDSF | Night Blue Vintage Coffee |
| VDSW | Black Vintage Coffee |

---

## 7. 차종 데이터 (어프로치북 추출 + 수동 정제)

> 37개 시리즈 / 약 150개 트림

| 시리즈 | 타입 | 주요 트림 예시 |
|--------|------|----------------|
| 1시리즈 | SEDAN | 120 Base / 120 M Sport / M135 xDrive |
| 2시리즈 Active Tourer | SEDAN | 218d Advantage ~ 220i M Sport DS |
| 2시리즈 Gran Coupe | SEDAN | 220 M Sport ~ M235 xDrive Pro |
| 3시리즈 | SEDAN | 320d Base ~ M340i Pro |
| 3시리즈 Touring | SEDAN | 320d Touring ~ M340i xD Touring Pro |
| 4시리즈 Coupe/Conv/GC | SEDAN | 420i ~ M440i xDrive |
| 5시리즈 | SEDAN | 523d ~ 550e xDrive M Sport Pro |
| 7시리즈 | SEDAN | 740d ~ 750e xDrive M Sport |
| 8시리즈 | SEDAN | M850i Coupe/GC xDrive M Perf |
| Z4 | SEDAN | Z4 sDrive 20i M Sport / M40i |
| M2 / M3 / M4 / M5 / M8 | SEDAN | Competition / Touring / Convertible |
| i4 / i5 / i7 | SEDAN | eDrive40 ~ M60/M70 |
| X1 / X2 / X3 / X4 | SUV | xLine ~ M Sport Pro |
| X5 / X6 / X7 | SUV | X-Line ~ M60i xDrive Pro |
| X3M / X4M / X5M / X6M | SUV | Competition |
| XM | SUV | XM Label |
| iX1 / iX2 / iX3 / iX | SUV | xLine ~ M70 xDrive |

---

## 8. 작업 플로우

### 일반 직원 — 차량 위치 확인
```
앱 접속
  → 구역별 마커 현황 즉시 확인
  → 마커 클릭 → 상세 정보 팝업 (외장/내장/번호/담당자/메모/위치)
```

### 일반 직원 — 차량 이동
```
🔒 이동 잠금 버튼 클릭
  → 🔓 이동 잠금 해제 상태로 전환
  → 마커 드래그 → 원하는 구역/슬롯에 드롭
  → Firebase 실시간 반영 → 전 직원 화면 즉시 업데이트
  → 🔓 버튼 다시 클릭 → 잠금 복귀
```

### 관리자 — 차량 추가
```
⚙️ 관리 버튼 → 비번 입력 (0123)
  → 관리자 바 활성화
  → + 차량 추가 클릭
  → 차량 구분 선택 (🚗 신차 / 🏎️ 시승차)
  → 시리즈 선택 → 트림 선택
  → 외장/내장 색상코드 선택
  → 차량번호/차대번호 입력
  → 담당자 입력
  → 위치(구역) 선택
  → 메모 입력 (선택)
  → 저장
```

### 관리자 — 구역 순서 변경
```
관리자 모드 활성화
  → 각 구역 헤더에 ⠿ 핸들 표시
  → 핸들 드래그 → 원하는 위치에 드롭
  → Firebase에 order값 교체 저장
```

### 관리자 — 구역 복구 (긴급)
```
관리자 모드 활성화
  → 🔄 구역 초기화 버튼 클릭
  → 확인 → 기본 9개 구역 재생성
  → 기존 마커 데이터 유지
```

---

## 9. 파일 구조

```
bmw_board.html          ← 단일 파일 앱 (전체 코드 포함)
bmw_models.txt          ← 차종 리스트 (수정용 원본)
BMW_GB_보드_프로젝트문서.md  ← 이 문서
```

---

## 10. 알려진 이슈 및 해결 이력

| 이슈 | 원인 | 해결 |
|------|------|------|
| 구역이 전혀 안 뜸 | `addZoneDrop`에서 `el.closest('.zone')` 호출 시점에 DOM 미연결 → null 에러로 renderBoard 중단 | `zoneEl`을 인자로 직접 전달하도록 수정 |
| 프라자 슬롯 미표시 | Firebase 기존 데이터에 `plaza` 필드 없음 | `zone.name === "프라자"` 조건 추가 |
| 마커 드롭이 구역 상단에서만 됨 | dragover 이벤트가 zone-body에만 등록됨 | zone 전체 요소에 이벤트 추가, `relatedTarget` 기반 dragleave 처리 |

---

## 11. 향후 개선 아이디어

- [ ] 차량 이동 이력 로그 (누가 언제 어디서 어디로)
- [ ] 구역별 차량 수 통계 / 현황 요약 헤더
- [ ] 차량 검색 기능 (차종명 / 차량번호)
- [ ] 입출고 날짜 기록
- [ ] 모바일 터치 드래그 정확도 개선
- [ ] 구역 삭제 기능 (관리자)
- [ ] 마커 색상 커스텀 테마

---

*Made by 남광희 · BMW 한독모터스 강북전시장*
