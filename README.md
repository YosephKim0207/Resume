<img width="150" alt="image" src="https://user-images.githubusercontent.com/46564046/235316172-496d7578-86f2-4d7f-85c2-d8bd62204b7f.jpg">

# 김요셉 | 게임 클라이언트 개발자

---

팀과 함께 만든 게임 속 세상이 플레이어에게 감동을 주길 바라는 클라이언트 개발자입니다.

플레이 가능한 알찬 게임 세상을 위해 오브젝트 풀링을 활용, 유니티 프로젝트 CPU 사용률 최대 50%, 언리얼 프로젝트 게임스레드 기준 13% 이상의 성능 향상. 길찾기 함수 호출을 최소화하면서 성능향상과 자연스러운 추적을 구현하였습니다.

스토리, 카메라 연출을 공부하고 돌리줌을 직접 적용해보는 등 유저 경험의 강화와 개발시 팀원간 소통을 위해 노력하고 있습니다.

**Github**: [https://github.com/YosephKim0207](https://github.com/YosephKim0207?tab=repositories)

**Email** : kjs940207@gmail.com

## Skills

---

### **Knowledge**

C++ / C# / Unreal / Unity

### **ETC**

Git 

## Game Projects

---

### 3인칭 루트슈터 게임 제작(개인과제 / C++ /  언리얼)

![EscapeFromAC_Default](https://user-images.githubusercontent.com/46564046/233114705-ffad50db-3746-49f2-94bb-7d93231bae29.gif)

기간 : 2023.02~2023.04 / 포트폴리오 링크 : https://github.com/YosephKim0207/EscapeFromAC

캐릭터의 모듈식 파츠 교체 시스템과 전투 시스템 구축

특징

- 모듈식 파츠를 통한 플레이어블 캐릭터의 외관 및 능력치 변화 구축
- UMG를 활용하여 플레이어블 캐릭터의 실시간 정보 및 파츠 변경에 따른 능력치 변화 시각화 UI 제작
- 사용자 편의성을 높인 오브젝트 풀링 구현을 통한 최적화
- 언리얼 자체 제공 함수를 활용한 전투 시스템 구축

### 탑다운 2D 슈팅게임 제작(개인과제 / C# / 유니티)

![플레이핵심완성2](https://user-images.githubusercontent.com/46564046/233091869-20c84a42-7a4b-41d9-8f8c-90f21aaeae48.gif)

기간 : 2022.05~2022.06 / 포트폴리오 링크 : https://github.com/YosephKim0207/Raiders

끊임 없이 플레이어를 쫓아오는 적과의 전투 시스템 구축

특징

- 투사체 발사 코드 개선을 통해 GC 및 Scripts의 CPU 사용량 최대 50% 향상
- 오브젝트 풀링을 통한 최적화
- A* 알고리즘을 응용한 길찾기 인공지능 제작

### 스토리 중심 게임 제작(학부개인과제 / C# / 유니티)

![전체보스연출](https://user-images.githubusercontent.com/46564046/235314111-4ea7c630-a4f5-4a8a-b34b-5859ca31969c.gif)

기간 : 2021.11 / 포트폴리오 링크 : https://github.com/YosephKim0207/MotionTest_Playable

단편 소설 집필 및 전체 게임 제작

특징

- 서사 중심 플레이 제작

- 카메라 워크 제작

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
- 투사체 내 연산의 최소화를 통해 원하는 투사체 발사 효과를 구현. 프로파일러 기준 GC 24%, Scripts 53% 개선




### 언리얼 프로젝트에서의 오브젝트 풀링 구현

문제
- 사용시 직관적이지 못한 웹 상에 공유되는 오브젝트 풀링 코드

원인
- 오브젝트 풀 이용을 원하는 경우 별도의 작업 없이 접근 및 사용을 희망

해결
- Class를 Key, 오브젝트 풀을 Value로 하는 TMap 형태의 PoolManager를 GameInstance에서 관리

- 템플릿을 통해 Class에 대한 오브젝트 풀을 조회 및 액터르 반환하는 함수 구현

- 구현된 오브젝트 풀링을 통해 프로파일러 기준  13% 개선



### 길찾기 알고리즘의 최적화

문제1

- A* 알고리즘의 F값이 급증하는문제

원인
- 타일맵 오브젝트의 collision 그리드의 영역 불일치로 인해 경로탐색 상 이동불가지역에 플레이어가 위치

해결
- 이동 불가지역 주변의 이동 가능 경로를 탐색하도록 하여 F값 급증 문제 해결 및 알고리즘 이용 객체들이 동일한 좌표를 향해 무리지어 이동하는 현상을 완화

문제2

- 적 숫자 증가에 따른 프레임 저하

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
