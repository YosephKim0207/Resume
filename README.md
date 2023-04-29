# 김요셉 | 게임 클라이언트 개발자

---

~~프론트엔드에서 세그먼트를 이용해 서버요청을 줄여 67% 이상 성능을 향상시켰으며 15배의 자료 증가가 되었지만 불필요한 로직과 요청을 제거해 30% 의 성능향상을 이끈 경험과 AI, IOT 등 다양한 프로젝트를 진행한 경험이 있다. Mocha.js 의 contributor이며 스타트업해커톤 1위, IOT 대회 2위, 백준 상위 0.1%의 실력을 가지고 있다.~~

**Github**: [https://github.com/YosephKim0207](https://github.com/YosephKim0207?tab=repositories)

**Email** : kjs940207@gmail.com

## Skills

---

C++ / C# / Unreal / Unity

### **ETC**

Git 

## Game Projects

---

### 3인칭 루트슈터 게임 제작(개인과제 / C++ /  언리얼)

![EscapeFromAC_Default](https://user-images.githubusercontent.com/46564046/233114705-ffad50db-3746-49f2-94bb-7d93231bae29.gif)

기간 : 2023.02~2023.04 / 링크 : https://github.com/YosephKim0207/EscapeFromAC

캐릭터의 모듈식 파츠 교체 시스템과 전투 시스템 구축

특징

- 모듈식 파츠를 통한 플레이어블 캐릭터의 외관 및 능력치 변화 구축
- UMG를 활용하여 플레이어블 캐릭터의 실시간 정보 및 파츠 변경에 따른 능력치 변화 시각화 UI 제작
- 오브젝트 풀링을 통한 최적화
- 언리얼 자체 제공 함수를 활용한 전투 시스템 구축

### 탑다운 2D 슈팅게임 제작(개인과제 / C# / 유니티)

![플레이핵심완성2](https://user-images.githubusercontent.com/46564046/233091869-20c84a42-7a4b-41d9-8f8c-90f21aaeae48.gif)

기간 : 2022.05~2022.06 / 링크 : https://github.com/YosephKim0207/Raiders

끊임 없이 플레이어를 쫓아오는 적과의 전투 시스템 구축

특징

- 투사체 발사 코드 개선을 통해 GC 및 Scripts의 CPU 사용량 최대 50% 향상
- 오브젝트 풀링을 통한 최적화
- A* 알고리즘을 응용한 길찾기 인공지능 제작

## ETC

---

### Watcher - 학생의 프로그래밍 활동 추적 시스템 (학부연구생과제 / 파이썬)

- Django를 통한 데이터 시각화 웹페이지 구축

## 개선 / 문제해결 사례

---


### 유니티 프로젝트에서의 많은 투사체 발사 반응속도 문제 및 성능 최적화

문제
- 순간적으로 다량의 투사체 발사시 의도와 다른 발사 조작감 발생

원인
- 투사체 생성에 불필요한 함수 사용 및 투사체 내부의 연산

해결
- 유니티 Documatation을 통해 보다 적절한 함수로 개선
- 오브젝트 풀링 적용
- 투사체 내 연산의 최소화를 통해 원하는 투사체 발사 효과를 구현함은 물론 프로파일러 기준 CPU사용률 50% 개선


### 길찾기 알고리즘의 최적화

문제1 : A* 알고리즘의 F값이 급증하는문제

원인
- 타일맵 오브젝트의 collision 그리드의 영역 불일치로 인해 경로탐색 상 이동불가지역에 플레이어가 위치

해결
- 이동 불가지역 주변의 이동 가능 경로를 탐색하도록 하여 F값 급증 문제 해결 및 알고리즘 이용 객체들이 동일한 좌표를 향해 무리지어 이동하는 현상을 완화

문제2 : 적 숫자 증가에 따른 프레임 저하

원인
- 다수의 적이 플레이어를 추적하기 위한 길찾기 알고리즘을 빈번하게 수행

해결
- 플레이어가 일정시간 동안 움직일 수 있는 거리는 한정되어 있으며 월드에 장애물의 밀도는 낮은 편. 따라서 플레이어 추적 및 이동에 단순 직선 이동과 조건에 따른 선택적 길찾기 알고리즘 호출을 조합


### UMG를 이용한 UI 구축 문제

문제
- UI에 원하는 기능을 추가하려는 경우 블루프린트의 대대적인 수정이 빈번하게 발생

원인
- 학습 중인 언리얼 UMG 튜토리얼 결과에 필요한 기능을 그때그때 추가하는 방식으로 UI 제작을 진행하여 해당 형태로는 원하는 기능 추가가 불가능

해결
- 구체적인 청사진을 통해 게임의 핵심 시스템 및 확장성을 염두하여 UI를 설계 및 다시 제작
