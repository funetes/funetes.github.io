---
layout: post
title: "Kiosk Project"
date: 2026-05-15
categories: Unity
tags:
  - unity
---

# Kiosk Project

## 기간

- 2026.05.13(수) ~ 05.15(금)

## 개발진행

[개발진행 문서](https://www.notion.so/35f3f8c547ec80678009e4eeb46288ce?source=copy_link)

[repository](https://github.com/funetes/0522-Kiosk-project)

---

### 1일차 (5월 13일): 아키텍처 정립 및 리팩토링

- **기본 프레임워크 수립 및 아키텍처 분리:** AI로 빠르게 기본 프로토타입을 빌드한 이후, 설계 개선 작업을 거쳤습니다. `CafeKioskController` 한곳에 뭉쳐있던 UI 생성, 제어, 비즈니스 로직을 깔끔하게 분리하기 위해 **MVVM 패턴 기반의 `ViewModel`로 분리**하였습니다.
- **UI 코드 모듈화:** 컨트롤러 내부의 UI 렌더링 코드를 별도의 클래스로 완전히 분리함으로써 코드의 단일 책임 원칙(SRP)을 지키고 가독성을 높였습니다.
- **레이아웃 및 주요 로직 개선:** 메뉴 목록 화면에서 하단 버튼이 잘려 보였던 레이아웃 버그를 수정했으며, 특정 옵션이 지정된 음료가 장바구니에 담기지 않는 중대 버그를 분석하여 해결했습니다.
- **협업을 위한 주석 작업:** 팀원들과의 코드 공유 및 원활한 병합(Merge)을 위해 핵심 로직 파일(`CafeKioskOrderScreen` 등)에 라인별 설명을 덧붙였습니다.

### 2일차 (5월 14일): 기능 확장 및 완성도 극대화 (UX/기능 연동)

- **이벤트 기반 옵션 처리:** `ViewModel` 내의 일시적인 옵션 선택 상태(`PendingOptionItem`)가 설정되는 시점에 맞추어 `OptionPopup`이 실시간으로 이벤트를 수신하도록 **C# Event Action 기반 반응형 결합**을 구성했습니다.
- **세밀한 비즈니스 로직 예외 처리:** 에이드(Ade)와 같은 상시 아이스 음료에는 'Hot' 선택 버튼이 제공되지 않도록 유기적인 예외 필터링 조건을 추가하고, 선택된 효과 디자인을 실시간으로 변경했습니다.
- **유연한 레이아웃 확장:** 다양한 결제 및 영수증 정보 출력을 유연하게 감당하도록 `Vertical Layout Group` 기반의 동적 UI 빌더(`Build2()`)를 설계했습니다.
- **결제/어드민 시스템 연동:** 멤버십 기반의 포인트 및 쿠폰 적립 기능(`MembershipService`)을 도입하고, 주문 완료 컨펌 및 영수증 팝업을 연계했습니다. 또한, 관리자 모드 비밀번호 검증과 정산 기능 팝업을 추가하였습니다.
- **디테일 마감 및 최적화:** 개발 과정에서 발견된 한글 인코딩 깨짐 문제를 해결하고, 중복 선언/함수 제거, 레이아웃 정밀 어긋남 조정 및 디버그용 로그들을 깔끔하게 정리하며 안정성을 확보했습니다.

---

## Flow

### Kiosk

<img src="https://github.com/user-attachments/assets/61ef570a-059d-4a73-a318-f5e4b82f652b" alt="키오스크 화면 흐름도 (Kiosk Flow)" width="100%" style="max-width: 1090px; display: block; margin: 1.5rem auto; border-radius: 8px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);" loading="lazy" />

### Admin

<img src="https://github.com/user-attachments/assets/60a83a30-3e25-4cfb-960e-2d43d87f4536" alt="관리자 화면 흐름도 (Admin Flow)" width="100%" style="max-width: 296px; display: block; margin: 1.5rem auto; border-radius: 8px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);" loading="lazy" />

### 시연

<video controls width="100%" style="max-width: 1090px; display: block; margin: 1.5rem auto; border-radius: 8px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);" preload="metadata">
  <source src="https://github.com/user-attachments/assets/b25c5828-def2-42e5-b177-a51e43371173" type="video/mp4" />
  사용 중이신 브라우저가 비디오 태그를 지원하지 않습니다. 대신 <a href="https://github.com/user-attachments/assets/b25c5828-def2-42e5-b177-a51e43371173">비디오 링크</a>를 클릭해 시청해 주세요.
</video>
