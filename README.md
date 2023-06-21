# 김요셉 | 게임 클라이언트 프로그래머

---

게임 속 경험을 통해 유저에게 감동을 주길 바라는 클라이언트 개발자입니다.

투사체 발사시 불필요한 내부 연산을 줄여 유니티 프로젝트 스크립트 CPU 사용률 50% 감소, 길찾기 함수 호출을 최적화하여 유니티 프로젝트 스크립트 CPU 사용률 80% 감소, 오브젝트 풀링을 활용하여 유니티 프로젝트 CPU 사용률 50% 감소, 언리얼 프로젝트 게임스레드 기준 13% 이상의 성능 향상을 달성하였습니다.

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

![EscapeFromAC_Final](https://github.com/YosephKim0207/Resume/assets/46564046/5185392a-2b47-4039-9736-62f4440b5dd6)

기간 : 2023.02~2023.04 / 포트폴리오 링크 : https://github.com/YosephKim0207/EscapeFromAC

캐릭터의 모듈식 파츠 교체 시스템과 전투 시스템 구축

특징

- 모듈식 파츠를 통한 플레이어블 캐릭터의 외관 및 능력치 변화 구축
- 개발 편의성을 높인 오브젝트 풀링 구현을 통한 게임 스레드 기준 13% 성능 향상
- UMG를 활용하여 플레이어블 캐릭터의 실시간 정보 및 파츠 변경에 따른 능력치 변화 시각화 UI 제작
- 위젯, 비헤이비어 트리 제외 99% C++ 구현

### 개선 / 문제해결 사례

### 언리얼 오브젝트 풀링


문제
- 전투시 성능 저하

원인
- 슈팅 게임으로서 빈번한 Projectile 액터 생성 및 제거로 인한 메모리 할당 / 해제

해결
- GameInstance에 오브젝트 풀 템플릿을 만들어 오브젝트 풀 이용을 원하는 클래스는 자유롭게 풀 생성 및 사용

결과
- 프로파일러 기준 평균 게임 스레드 성능 13% 향상



[PoolManager 전체 코드 바로가기](https://github.com/YosephKim0207/EscapeFromAC/blob/main/Source/EscapeFromAC/A_PoolManager.h)
<details>
<summary>PoolManager 구현 템플릿 코드 펼치기</summary>



```cpp
/**
 * Get template class's Actor from Object Pool
 * If Pool is not exist, Make object pool about class and return
 */
template <class T>
T* AA_PoolManager::GetThisObject(UClass* Class, const FVector& Location, const FRotator& Rotation,
	const FActorSpawnParameters& SpawnParameters)
{
	TArray<AActor*>* Pool = ObjectPool.Find(Class);
	
	// Object Pool is Exist
	if(Pool != nullptr)
	{
		UE_LOG(LogTemp, Log, TEXT("PoolManager : %s has Pool"), *Class->GetName());
		
		for(AActor* PoolingActor : *Pool)
		{
			bool IsHidden = PoolingActor->IsHidden();
		
			if(IsHidden)
			{
				PoolingActor->SetActorLocation(Location);
				PoolingActor->SetActorRotation(Rotation);
				PoolingActor->SetOwner(SpawnParameters.Owner);
				PoolingActor->SetInstigator(SpawnParameters.Instigator);
				PoolingActor->SetActorHiddenInGame(false);
		
				return Cast<T>(PoolingActor);
			}
		}

		UE_LOG(LogTemp, Warning, TEXT("PoolManager : %s Pool is empty"), *Class->GetName());
		
		return nullptr;
	}

	// Pool doesn't Exist
	UE_LOG(LogTemp, Warning, TEXT("PoolManager : %s doesn't have Pool"), *Class->GetName());
	
	TArray<AActor*> NewPool;
	UWorld* World = GetWorld();
	if(World)
	{
		for(int8 SpawnIndex = 0; SpawnIndex < PoolSize; ++SpawnIndex)
		{
			T* RespawnActor;
			RespawnActor = GetWorld()->SpawnActor<T>(Class, FVector::ZeroVector, FRotator::ZeroRotator);
			
			RespawnActor->SetActorHiddenInGame(true);
			NewPool.Add(RespawnActor);
		}
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("PoolManager : World is nullPtr"));

		return nullptr;
	}
	

	UE_LOG(LogTemp, Log, TEXT("PoolManager : %s make Pool"), *Class->GetName());
	
	ObjectPool.Add(Class, NewPool);
	
	AActor* ReturnActor;
	ReturnActor = (*ObjectPool.Find(Class))[0];
	ReturnActor->SetActorLocation(Location);
	ReturnActor->SetActorRotation(Rotation);
	ReturnActor->SetOwner(SpawnParameters.Owner);
	ReturnActor->SetInstigator(SpawnParameters.Instigator);
	ReturnActor->SetActorHiddenInGame(false);

	UE_LOG(LogTemp, Log, TEXT("PoolManager : return %s from init pool"), *Class->GetName());
	
	return Cast<T>(ReturnActor);
}
```

</details>
	
	
	
[A_Character 전체 코드 바로가기](https://github.com/YosephKim0207/EscapeFromAC/blob/main/Source/EscapeFromAC/A_Character.cpp)
<details>
<summary>A_Character에서의 PoolManager 사용 코드 펼치기</summary>



```cpp
if(GetbIsShootable() && GetbIsRightArmOnFire())
	{
		FActorSpawnParameters SpawnParameters;
		SpawnParameters.Owner = this;
		SpawnParameters.Instigator = GetInstigator();

		UWorld* World = GetWorld();
		UACGameInstance* ACGameInstance = Cast<UACGameInstance>(UGameplayStatics::GetGameInstance(GetWorld()));
		AA_PoolManager* PoolManager = ACGameInstance->GetPoolManager();
		if(PoolManager)
		{
			AA_Projectile* Projectile = PoolManager->GetThisObject<AA_Projectile>(TempProjectile, ProjectileRespawnLocation + (ProjectileShootRotator.Vector() * 10.0f), ProjectileShootRotator, SpawnParameters);

			if(Projectile != nullptr)
			{
				SetProjectileData(Projectile, EModular::ERightArm, true);
			}
			else
			{
				UE_LOG(LogTemp, Log, TEXT("%s : %s Pool is empty! Do SpawnActor"), *GetName(), *TempProjectile->GetName());
				Projectile = World->SpawnActor<AA_Projectile>(TempProjectile, ProjectileRespawnLocation + (ProjectileShootRotator.Vector() * 10.0f), ProjectileShootRotator, SpawnParameters);
				if(Projectile != nullptr)
				{
					SetProjectileData(Projectile, EModular::ERightArm, false);
				}
				else
				{
					UE_LOG(LogTemp, Warning, TEXT("%s : Get Projectile Fail!"), *GetName());
				}
			}
		}
		else
		{
			UE_LOG(LogTemp, Warning, TEXT("%s : PoolManager is nullPtr!"), *GetName());
		}
	}
```

</details>


### 탑다운 2D 슈팅게임 제작(개인과제 / C# / 유니티)

![플레이핵심완성2](https://user-images.githubusercontent.com/46564046/233091869-20c84a42-7a4b-41d9-8f8c-90f21aaeae48.gif)

기간 : 2022.05~2022.06 / 포트폴리오 링크 : https://github.com/YosephKim0207/Raiders

끊임 없이 플레이어를 쫓아오는 적과의 전투 시스템 구축

특징

- 투사체 발사 코드 개선을 통해 GC 및 Scripts의 CPU 사용량 50% 향상
- 길찾기 함수 호출 최적화를 통한 스크립트 CPU 사용률 80% 감소
- 오브젝트 풀링을 통한 CPU 사용률 50% 감소
- A* 알고리즘을 응용한 길찾기 인공지능 제작

### 개선 / 문제해결 사례

### 길찾기 함수 콜 최적화  

문제
- 다수의 NPC 출현시 성능 저하

원인
- 모든 NPC가 주기적으로 길찾기 함수를 호출하여 성능 부담

해결
- 전투 시스템 및 레벨 디자인 방향성을 고려하여 장애물 발견시에만 길찾기 함수 호출
- 캐릭터의 이동속도가 느린 점을 이용하여 일정 시간 동안 기존의 경로Stack 재사용
- 코루틴 사용 

결과
- 스크립트 CPU 사용률 80% 감소

[EnemyController 전체 코드 바로가기](https://github.com/YosephKim0207/Raiders/blob/main/Assets/Scripts/Controller/EnemyController.cs) /
[MapManager 전체 코드 바로가기](https://github.com/YosephKim0207/Raiders/blob/main/Assets/Scripts/Manager/MapManager.cs)

<details>
<summary>길찾기 코루틴 코드만 펼치기</summary>



```csharp
 // 플레이어를 타겟으로 길을 찾는 코루틴
    IEnumerator CoFindPath() {
        // 플레이어와 일정 거리 이내인 경우 다음 길찾기 중단
        if ((_shootTargetTransform.position - transform.position).magnitude < 2.5f) {
            State = CreatureState.Attack;
        }

        // 진행방향 전방 충돌 점검
        RaycastHit2D rayHit;
        rayHit = Physics2D.Raycast(this.transform.position, DestPos, 2.5f, LayerMask.GetMask("Collision"));
   
        Debug.DrawRay(this.transform.position, DestPos * 1.5f, Color.red, 1.0f);

        // Enemy가 길찾기에 사용할 조건 설정
        // rayHit1에 충돌한 오브젝트가 없고 && Manager의 FindPath로 탐색해둔 경로가 없으면 직선이동
        if (rayHit.transform == null && _pathStack == null) {

            PathState = FindPathState.UseDirect;

        }
        // Manager의 FindPath로 탐색해둔 경로가 _pathStack상에 없거나 || _pathStack가 비어있으면Manager의 FindPath를 호출하기 위해 PathStated를 ReFindPath로
        else if (_pathStack == null || _pathStack.Count == 0) {
            PathState = FindPathState.ReFindPath;
            }
        // Manager의 FindPath를 통해 찾은 경로를 정해진 횟수 이상 사용한 경우 경로 재탐색 및 사용횟수 초기화
        else if (usePathStackCount > pathStackUsageCount) {
            PathState = FindPathState.ReFindPath;
            usePathStackCount = 0;
            }
        // Manager의 FindPath로 찾아둔 경로를 _pathStack에서 가져와 사용
         else {
            PathState = FindPathState.UsePathStack;
         }

        // 정해진 조건대로 분기하여 이동경로 설정
        switch (PathState) {
            case FindPathState.UseDirect:
                DestPos = (_shootTargetTransform.position - transform.position).normalized;
                break;
            case FindPathState.ReFindPath:
                _pathStack = Manager.Map.FindPath(this.transform, _shootTargetTransform);
                SetPathUseStack();
                break;
            case FindPathState.UsePathStack:
                SetPathUseStack();
                ++usePathStackCount;
                break;
        }

        yield return WaitFindPathTime;

        _coFindPath = null;
    }

    // _pathStack에 저장된 경로를 이용하여 경로 설정 
    void SetPathUseStack() {
        Vector3 nextPos;
        nextPos = _pathStack.Pop();
        if ((_pathStack.Count > 0) && (nextPos - transform.position).magnitude < 0.5) {
            nextPos = _pathStack.Pop();
        }
        DestPos = (nextPos - transform.position).normalized;
    }
```

</details>


### 유니티 프로젝트에서의 많은 투사체 발사 성능 최적화

문제
- 다수의 총알이 발사되는 Fever 스킬 사용시 및 다수의 NPC가 사격시 성능 저하

원인
- 총알을 발사하는 캐릭터 판정 및 충돌 판정에 대한 불필요한 정보 전달
- BulletController의 Update함수 내부에 존재하는 불필요한 연산

해결
- 오브젝트 풀에서 총알 호출 시 총알의 Transform과 총알을 생성하는 클래스만 전달
- Update함수에서는 총알이 카메라를 벗어났는지만 확인

결과
- 스크립트 CPU 사용률 50% 감소

[BulletController 전체 코드 바로가기](https://github.com/YosephKim0207/Raiders/blob/main/Assets/Scripts/Controller/BulletController.cs)
<details>
<summary>BulletController 전체 코드 펼치기</summary>



```csharp
public class BulletController : MonoBehaviour {
    public CreatureController SetCreature { set => _creature = value; }
    public Vector3 DestPos { get; set; }

    Rigidbody2D _rigidbody;
    Vector3 _bullPos;
    Camera _cam;
    CreatureController _creature;
    const float _outRange = 0.05f;

    private void Awake() {
        _rigidbody = GetComponent<Rigidbody2D>();
        _cam = Camera.main;
    }

    private void OnEnable() {
        float speed = 28.0f;
        _rigidbody.velocity = DestPos * speed;
    }


    protected virtual void Update() {
        // 카메라 영역+a 벗어나는 경우 총알 제거
        _bullPos = _cam.WorldToViewportPoint(transform.position);
        if (_bullPos.x <= -_outRange || _bullPos.x >= 1.0f + _outRange || _bullPos.y <= -_outRange || _bullPos.y >= 1.0f + _outRange) {
            _creature = null;
            Manager.Pool.PushPoolChild(this.gameObject);
        }
    }

    
    private void OnTriggerEnter2D(Collider2D collision) {
        // 총알 간의 충돌이 발생하는 경우 || 아이템과 충돌하는 경우 
        if (collision.gameObject.layer.Equals(8) || collision.gameObject.layer.Equals(9)) {
            return;
        }
        // Player가 쏜 총알이 Enemy와 충돌하는 경우
        else if (collision.gameObject.layer.Equals(10)) {
            if (_creature is PlayerController) {
                EnemyController enemy = collision.GetComponent<EnemyController>();

                if (enemy != null) {
                    enemy.HP -= _creature.GunInfo.damage;
                    _creature = null;
                    Manager.Pool.PushPoolChild(gameObject);

                    Debug.Log("Player's Bullet Hit Enemy");
                }
            }
        }
        // Enemy가 쏜 총알이 Player와 충돌하는 경우
        else if (collision.gameObject.layer.Equals(6)) {
            if (_creature is EnemyController) {
                PlayerController player = collision.GetComponent<PlayerController>();
                // player Fever인 경우 무적
                if (player.IsFever == false) {
                    player.HP -= _creature.GunInfo.damage;

                    Debug.Log("Enemy's Bullet Hit Player");
                }

                _creature = null;
                Manager.Pool.PushPoolChild(gameObject);
            }
        }
        else {
            Manager.Pool.PushPoolChild(gameObject);
        }

        
    }
}
```

</details>


### 유니티 오브젝트 풀링
 
문제
- 다수의 NPC가 사격시 성능 저하

원인
- Bullet 객체가 빈번하게 생성 및 소멸하여 메모리 할당 / 해제, GC, 메모리 파편화 문제 발생
- Bullet 객체뿐만 아니라 NPC의 잦은 생성 및 소멸도 문제

해결
- 오브젝트 풀링 구현

결과
- CPU 사용률 50% 감소

[PoolManager 전체 코드 바로가기](https://github.com/YosephKim0207/Raiders/blob/main/Assets/Scripts/Manager/PoolManager.cs)
<details>
<summary>PoolManager 전체 코드 펼치기</summary>



```csharp
public class PoolManager {

    class Pool {
        Stack _poolStack = new Stack();

        // initNum만큼 Pool에 child Push하는 초기화 진행
        public void Init(GameObject originObj, GameObject parentObj, int initNum = 5) {
            for (int i = 0; i < initNum; ++i) {
                GameObject childObj = Object.Instantiate(originObj, parentObj.transform);
                childObj.name = $"{originObj.name}";
                childObj.SetActive(false);
                _poolStack.Push(childObj);
            }
        }

        public void Push(GameObject originObj, GameObject parentObj = null) {
            originObj.SetActive(false);
            _poolStack.Push(originObj);
        }

        public GameObject Pop(Vector3 pos, Quaternion rot) {
            GameObject childObj = (GameObject)_poolStack.Pop();

            childObj.transform.position = pos;
            childObj.transform.rotation = rot;

            childObj.SetActive(true);
            return childObj;
        }

        public GameObject Pop(Vector3 pos, Quaternion rot, Vector3 shootDir) {
            GameObject childObj = (GameObject)_poolStack.Pop();

            childObj.transform.position = pos;
            childObj.transform.rotation = rot;
            
            BulletController bullet = childObj.GetComponent<BulletController>();
           
            bullet.DestPos = shootDir;

            childObj.SetActive(true);
            return childObj;
        }

        public GameObject Pop(Vector3 pos, Quaternion rot, Vector3 shootDir, CreatureController creatureCS) {
            GameObject childObj = (GameObject)_poolStack.Pop();

            childObj.transform.position = pos;
            childObj.transform.rotation = rot;

            BulletController bullet = childObj.GetComponent<BulletController>();
            bullet.DestPos = shootDir;
            bullet.SetCreature = creatureCS;

            childObj.SetActive(true);
            return childObj;
        }

        public int Count() {
            return _poolStack.Count;
        }
    }

    // Obj당 Pool들 관리
    Dictionary<string, Pool> _poolDic = new Dictionary<string, Pool>();    
    GameObject _root;

    public void Init() {
        _root = GameObject.Find("@RootPool");

        if(_root == null) {
            _root = new GameObject("@RootPool");
        }
    }

    public GameObject UsePool(GameObject originObj) { 
        Transform poolObj = _root.transform.Find(originObj.name);
        if (poolObj == null) {
            // Pool들의 root 하위로 originObj에 대한 pool 생성
            poolObj = InitPoolObj(originObj, _root.transform).transform;

            // return PopPoolChild(originObj, poolObj);
            return PopPoolChild(originObj, poolObj.gameObject, Vector3.zero, Quaternion.identity);
        }
        else {
            // return PopPoolChild(originObj, poolObj);
            return PopPoolChild(originObj, poolObj.gameObject, Vector3.zero, Quaternion.identity);
        }
    }
    public GameObject UsePool(GameObject originObj, Vector3 pos, Quaternion rot, Vector3 shootDir) {
        //GameObject poolObj = GameObject.Find(originObj.name);
        Transform poolObj = _root.transform.Find(originObj.name);
        if (poolObj == null) {
            // Pool들의 root 하위로 originObj에 대한 pool 생성
            poolObj = InitPoolObj(originObj, _root.transform).transform;

            // return PopPoolChild(originObj, poolObj);
            return PopPoolChild(originObj, poolObj.gameObject, pos, rot, shootDir);
        }
        else {
            // return PopPoolChild(originObj, poolObj);
            return PopPoolChild(originObj, poolObj.gameObject, pos, rot, shootDir);
        }
    }

    public GameObject UsePool(CreatureController creatureSC, GameObject originObj, Vector3 pos, Quaternion rot, Vector3 shootDir) {
        Transform poolObj = _root.transform.Find(originObj.name);
        if (poolObj == null) {
            // Pool들의 root 하위로 originObj에 대한 pool 생성
            poolObj = InitPoolObj(originObj, _root.transform).transform;

            return PopPoolChild(creatureSC, originObj, poolObj.gameObject, pos, rot, shootDir);
        }
        else {
            return PopPoolChild(creatureSC, originObj, poolObj.gameObject, pos, rot, shootDir);
        }
    }

    GameObject InitPoolObj(GameObject originObj, Transform parent) {
        // Pooling된 Obj들이 들어갈 부모 Obj 만들기
        GameObject poolObj = new GameObject(name : originObj.name);
        poolObj.transform.parent = parent;

        // 새로 생긴 poolObj 초기화
        Pool pool = new Pool();
        pool.Init(originObj, poolObj);

        // 원본obj이름을 key로 딕셔너리에 Stack add
        _poolDic.Add(originObj.name, pool);

        return poolObj;
    }

    GameObject PopPoolChild(GameObject originObj, GameObject parentObj, Vector3 pos, Quaternion rot) {
        // 해당 오브젝트의 pool이 없는 경우
        if (_poolDic.ContainsKey(originObj.name).Equals(false)) {
            //Debug.Log("PopPoolChild_ContainsKey : True");

            Pool pool = new Pool();
            pool.Init(originObj, parentObj);
            _poolDic.Add(originObj.name, pool);
            return pool.Pop(pos, rot);
        }
        // 해당 오브젝트의 pool이 있는 경우 
        else {
            Pool pool = _poolDic[originObj.name];

            // 비활성화 된 Obj가 남아있는 경우
            if (!pool.Count().Equals(0)) {
                //return pool.Pop();
                return pool.Pop(pos, rot);
            }
            // 비활성화 된 Obj가 남아있지 않은 경우
            else {
                // Obj를 Pool에 추가 생성(Init을 이용) 및 Pop
                pool.Init(originObj, parentObj, 1);
                //return pool.Pop();
                return pool.Pop(pos, rot);
            }
        }
    }

    // Pool에서 Obj pop하여 사용하기 
    GameObject PopPoolChild(CreatureController creatureCS, GameObject originObj, GameObject parentObj, Vector3 pos, Quaternion rot, Vector3 shootDir) {
        // 해당 오브젝트의 pool이 없는 경우
        if (_poolDic.ContainsKey(originObj.name).Equals(false)) {
            //Debug.Log("PopPoolChild_ContainsKey : True");

        Pool pool = new Pool();
        pool.Init(originObj, parentObj);
        _poolDic.Add(originObj.name, pool);
        return pool.Pop(pos, rot, shootDir, creatureCS);

        }
        // 해당 오브젝트의 pool이 있는 경우 
        else {
            Pool pool = _poolDic[originObj.name];

            // 비활성화 된 Obj가 남아있는 경우
            if (!pool.Count().Equals(0)) {
                return pool.Pop(pos, rot, shootDir, creatureCS);
            }
            // 비활성화 된 Obj가 남아있지 않은 경우
            else {
                // Obj를 Pool에 추가 생성(Init을 이용) 및 Pop
                pool.Init(originObj, parentObj, 1);
                return pool.Pop(pos, rot, shootDir, creatureCS);
            }
        }
    }

    // Pool에서 Obj pop하여 사용하기 
    GameObject PopPoolChild(GameObject originObj, GameObject parentObj, Vector3 pos, Quaternion rot, Vector3 shootDir) {
        // 해당 오브젝트의 pool이 없는 경우
        if (_poolDic.ContainsKey(originObj.name).Equals(false)) {
            Debug.Log("PopPoolChild_ContainsKey : True");

            Pool pool = new Pool();
            pool.Init(originObj, parentObj);
            _poolDic.Add(originObj.name, pool);
            return pool.Pop(pos, rot, shootDir);

        }
        // 해당 오브젝트의 pool이 있는 경우 
        else {
            Pool pool = _poolDic[originObj.name];

            // 비활성화 된 Obj가 남아있는 경우
            if (!pool.Count().Equals(0)) {
                return pool.Pop(pos, rot, shootDir);
            }
            // 비활성화 된 Obj가 남아있지 않은 경우
            else {
                // Obj를 Pool에 추가 생성(Init을 이용) 및 Pop
                pool.Init(originObj, parentObj, 1);
                //return pool.Pop();
                return pool.Pop(pos, rot, shootDir);
            }
        }
    }

    // 사용한 오브젝트를 PoolRoot에 반환 
    public void PushPoolChild(GameObject childObj) {
        Pool pool = _poolDic[childObj.name];
        childObj.SetActive(false);
        pool.Push(childObj);
    }

    // Pool 제거
    public void DeletePool(GameObject originObj, Transform parent = null) {
        if(parent == null) {
            GameObject poolObj = GameObject.Find(originObj.name);
            Object.Destroy(poolObj);
            _poolDic.Remove(originObj.name);
        }
        else {
            // TODO
            // parent를 이용하는 경우 추가하기
            Debug.Log("DeletePool Use parent need TODO");
        }
    }

}
```

</details>


### 플레이어의 특정 위치 진입시 프리징 문제

문제
- 간헐적으로 NPC가 경로탐색시 사용하는 A* 알고리즘의 F값이 급증하는 문제

원인
- 장애물의 타일맵 collider가 그리드 영역보다 작아 NPC가 맵을 인식하는 데이터 상의 이동 불가 지역에 플레이어가 진입

해결
- 단순히 콜리전의 모양을 변경하는 것은 이후에도 비슷한 문제 발생 가능
- 이동 불가지역 주변의 이동 가능 경로를 탐색하는 기능 추가
 
결과
- F값 급증 문제 해결 및 알고리즘 이용 객체들이 동일한 좌표를 향해 무리지어 이동하는 현상을 완화


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

기간 : 2020.08~2021.12

- Django를 통한 데이터 시각화 웹페이지 구축

### 소켓 프로그래밍 (학부과제 / C)

소스코드 링크 : https://github.com/YosephKim0207/SocketProgramming

- 소켓 통신을 통한 파일 송수신

 <img width="600" alt="image" src="https://user-images.githubusercontent.com/46564046/235702302-f454d4f1-eb2c-49e0-84fa-75a2303e197a.png">



