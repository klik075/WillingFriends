# 🐔 WillingFriends

> 길건너 친구들에 전투를 더한 **3D 복셀 쿼터뷰 액션 게임**. 차와 강을 피해 앞으로 나아가며 몬스터를 물리칩니다.
> 4인 팀에서 **오브젝트 풀링, 사운드 매니저, 무기 드롭 아이템**을 맡았습니다.

[![WillingFriends 시연 영상](https://img.youtube.com/vi/RKIOK1jr8oU/hqdefault.jpg)](https://youtu.be/RKIOK1jr8oU)

**[▶️ 시연 영상](https://youtu.be/RKIOK1jr8oU)** · 원본 팀 저장소 [iou-bohun/WillingFriends](https://github.com/iou-bohun/WillingFriends)

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 기간 / 인원 | 2024.02.26 \~ 03.05 (약 1주) / 4명 · 내일배움캠프 팀 과제 |
| 장르 / 플랫폼 | 3D 복셀 쿼터뷰 액션 / PC |
| 내 역할 | 오브젝트 풀링, 사운드 매니저, 무기 드롭 아이템, 싱글톤 기반 클래스 |
| 기술 스택 | **Unity 2022.3 · C#** |
| 내 기여 | 커밋 41개 · 스크립트 7개 (약 490줄) |

## 내가 해결한 것

**1. 태그 기반 공용 오브젝트 풀**
- 태그별 `Queue`를 딕셔너리로 관리하고, 비어 있으면 새로 만들어 채우는 `ObjectPoolManager`
- 몬스터·무기·맵 발판·드롭 아이템 등 팀원 코드를 포함한 13개 스크립트에서 같은 풀을 사용

**2. 풀링 기반 사운드 매니저**
- `AudioSource` 오브젝트를 풀에서 꺼내 재생하고, 재생이 끝나면 스스로 반환
- 부모 오브젝트가 풀로 반환될 때 붙어 있던 사운드도 함께 반환되도록 구조 수정

---

## 상세 내용

<details>
<summary><b>🎲 게임 소개</b></summary>

1. 방향키로 한 칸씩 점프하며 앞으로 나아갑니다.
2. 도로의 자동차와 강을 피하고, 길을 막는 몬스터는 공격해서 처치합니다.
3. 몬스터를 처치하면 무기가 떨어지고, 주우면 공격 방식이 바뀝니다.
4. 몬스터에 부딪히거나, 강에 빠지거나, 시간이 다 되면 게임 오버입니다.

</details>

<details>
<summary><b>🛠 주요 구현</b></summary>

📁 `Assets/JWWFolder/Scripts/`

### 1. ObjectPoolManager

- 인스펙터에 `태그 / 프리팹 / 개수`를 등록하면 시작 시 미리 생성합니다.
- `GetObject(tag)`: 큐에 남은 객체가 있으면 꺼내고, 없으면 새로 생성해 반환
- `ReturnObject(tag, obj)`: 비활성화 후 해당 태그 큐로 반환

### 2. SoundManager / AudioSourceObject

- 효과음 재생 요청이 오면 풀에서 `AudioSourceObject`를 꺼내 위치를 잡고 재생합니다.
- `AudioSourceObject`는 재생이 끝나면 스스로 풀로 돌아갑니다.

### 3. 무기 드롭 아이템

- 몬스터가 죽으면 `DropItemManager`가 무기 데이터(`AttackSO`) 중 하나를 골라 풀에서 아이템을 꺼냅니다.
- 플레이어가 주우면 공격 데이터를 교체하고, 줍지 않으면 10초 뒤 사라집니다.

### 4. SingletoneBase\<T\>

- 매니저 클래스들이 상속하는 제네릭 싱글톤 기반 클래스

</details>

<details>
<summary><b>🔥 트러블슈팅</b></summary>

**1. 몬스터와 함께 사라지는 사운드 오브젝트**
- 문제: 소리를 낸 오브젝트의 자식으로 `AudioSourceObject`를 붙였더니, 몬스터가 풀로 반환될 때 사운드 오브젝트도 함께 비활성화되어 사운드 풀로 돌아오지 않음
- 해결: `ReturnObject`에서 반환되는 오브젝트의 자식 `AudioSourceObject`를 먼저 사운드 풀로 반환하고, 재생 위치는 부모 위치로 맞추도록 수정

**2. 플레이어 사망 소리가 들리지 않는 문제**
- 문제: 사망한 플레이어를 부모로 지정해 소리를 붙이면 플레이어와 함께 소리도 꺼짐
- 해결: 부모 없이 재생할 수 있도록 `parent`를 선택 인자로 바꾸고 null 처리 추가

</details>

<details>
<summary><b>💭 회고</b></summary>

**잘한 점**
- 팀 전체가 쓰는 풀을 먼저 만들어 각자 `Instantiate`/`Destroy`로 구현하지 않게 함

**아쉬운 점**
- 문자열 태그 기반이라 오타·이름 변경에 취약 → enum이나 프리팹 참조를 키로 쓰는 편이 안전
- 아이템의 사라짐 타이머를 `Start`의 `Invoke`로 걸어, 풀에서 재사용될 때 타이머가 다시 걸리지 않을 수 있음 → `OnEnable`에서 설정해야 함

</details>

## 팀 구성

| 이름 | 담당 | GitHub |
| --- | --- | --- |
| 최보훈 (팀장) | 몬스터 로직, UI | [iou-bohun](https://github.com/iou-bohun) |
| 국지윤 | 맵 생성(구조물 배치), 에셋 제작 | [JY-LemongO](https://github.com/JY-LemongO) |
| 박소이 | 플레이어 이동, 플레이어 아이템 | [ruiroll](https://github.com/ruiroll) |
| 정원우 | 오브젝트 풀링, 사운드, 무기 드롭 | [klik075](https://github.com/klik075) |
