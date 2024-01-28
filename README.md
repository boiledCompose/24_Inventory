## Room

Room은 SQLite 위에 있는 추상화 레이어로, 내부 데이터베이스를 구현하는데 사용하는 Jetpack 라이브러리다.

Kotlin은 데이터 클래스를 사용하여 메모리 내 데이터로 쉽게 작업할 수 있는 방법을 제공한다.    
허나 데이터 유지와 관련해선 이 데이터를 데이터베이스 저장소와 호환되는 형식으로 변환해야 한다. 이렇게 하려면 *데이터를 저장할 테이블*과 *데이터에 액세스하고 데이터를 수정할 쿼리*가 있어야 한다.

다음과 같은 세 가지의 Room 구성요소를 통해 이러한 워크플로가 원활해진다.

- Room 항목은 앱 데이터베이스와 테이블을 나타낸다. 테이블의 행에 저장된 데이터를 업데이트하고 삽입할 새 항을 만드는 데 사용한다.
- Room DAO는 앱이 데이터베이스에서 데이터를 검색, 업데이트, 삽입, 삭제하는 데 사용하는 메서드를 제공한다.
- Room Database 클래스는 DAO 인스턴스를 앱에 제공하는 클래스이다.

<br>

### Room 종속 항목

`build.gradle.kts (Module: app)`의 `dependecies` 블록에 다음 코드를 추가한다.
```kotlin
//Room
implementation("androidx.room:room-runtime:${rootProject.extra["room_version"]}")
ksp("androidx.room:room-compiler:${rootProject.extra["room_version"]}")
implementation("androidx.room:room-ktx:${rootProject.extra["room_version"]}")
```
>[!NOTE]
> Gradle 파일에 포함하는 라이브러리 종속 항목의 경우 항상 [AnroidX 출시](https://developer.android.com/jetpack/androidx/versions?hl=ko) 페이지에 표시된 최신 안정화 출시 버전 번호를 사용하도록 한다.

<br>

## Entity

Entity 클래스는 테이블을 정의하고 이 클래스의 각 인스턴스는 데이터베이스 테이블의 행을 나타낸다.

### Entity 클래스 만들기

1. 테이블의 엔티티로 만들고자 하는 데이터 클래스를 생성한다.
2. `@Entity` 주석을 달고 `tableName` 인수를 사용하여 테이블의 이름을 설정한다.
3. `@Primary` 주석으로 기본 키를 설정한다. 기본 키는 데이터베이스 테이블의 모든 레코드/항목을 고유하게 식별하는 데 사용된다. 앱이 기본 키를 할당한 후에는 수정할 수 없다.
4. 기본 키에 기본 값을 할당한다. 또한 주석의 `autoGenerate` 인수에 `true` 값을 넘겨 엔티티 생성 시 기본 값부터 자동으로 키를 부여하도록 설정한다.
   ```kotlin
   //예제 코드
   @Entity(tableName = "items")
   data class Item(
       @PrimaryKey(autoGenerate = true)
       val id: Int,
       val name: String,
       val price: Double,
       val quantity: Int
   )
   ```

## DAO

DAO는 애플리케이션의 데이터베이스 작업 실행과 관련된 모든 복잡성을 숨긴다. 이를 통해 데이터를 사용하는 코드와 관계없이 데이터 레이어를 변경할 수 있다.

안드로이드에서 DAO는 데이터베이스에 액세스하는 인터페이스를 정의하는 Room의 기본 구성요소로, 데이터베이스 쿼리/검색, 삽입, 삭제, 업데이트를 위한 편의 메서드를 제공한다.   이러한 기능들은 `@Insert`, `@Delete`, `@Update`, `@Query`와 같은 편의성 주석을 제공한다.

안드로이드 스튜디오는 쿼리를 작성할 때 컴파일러가 SQL 쿼리에 문법 오류가 있는지 확인해준다.

### Dao 인터페이스 만들기

1. Dao 인터페이스 파일을 생성한다. 예) `ItemDao.kt`
2. 인터페이스에 `@Dao` 주석을 단다.
   ```kotlin
   //예제 코드
   @Dao
   interface ItemDao {
   }
   ```

**Insert 메서드:**
1. 엔티티 삽입을 위한 메서드을 만드려면, 메서드에 `@Insert` 주석을 단다.
2. 함수를 정지 함수로 선언하여 별도의 스레드에서 실행되도록 한다.
3. 데이터베이스에 항목을 삽입할 때 같은 엔티티가 삽입되는 `충돌`이 발생할 수 있다. 이러한 충돌을 해결하기 위해 `onConflict` 인수를 추가하여 충돌 발생 시 Room이 실행할 작업을 지정한다. 이 예제의 경우 새로운 엔티티를 무시하는 `onConflictStrategy.IGNORE`을 사용한다.
   ```kotlin
   //예제 코드
   @Insert(onConflict = OnConflictStrategy.IGNORE)
   suspend fun insert(item: Item)
   ```

**Update 메서드**
```kotlin
@Update
suspend fun update(item: Item)
```

**Delete 메서드**
```kotlin
@Delete
suspend fun delete(item: Item)
```

**Query 메서드**
- 삽입, 업데이트, 삭제 이외의 기능들은 편의 주석이 없으므로 `@Query` 주석을 통해 SQL 쿼리를 사용하여 데이터베이스에 접근한다.
- 쿼리문 안에선 콜론 표기법을 사용하여 함수의 인수를 사용한다.

```kotlin
@Query("SELECT * from items WHERE id= :id)
fun getItem(id: Int): Flow<Item>
```

>[!NOTE]
> 지속성 레이어인 `Dao`에선 `Flow`를 사용하는 것이 좋다.
> - `Flow`를 반환 유형으로 사용하여 데이터베이스의 데이터가 변경될 때마다 알림을 받는다.
> - `Room`에선 `Flow`를 자동으로 업데이트하므로 명시적으로 한 번만 데이터를 가져오면 된다.
> - `Flow` 반환 유형으로 인해 Room은 백그라운드 스레드에서도 쿼리를 실행할 수 있다.
<br>

## Database

`Database` 클래스는 정의된 DAO의 인스턴스를 앱에 제공한다. 앱은 제공받은 DAO를 사용하여 데이터베이스의 데이터를 연결된 데이터 항목 객체의 인스턴스로 검색할 수 있다.

### 데이터베이스 만들기

1. `Database` 클래스 파일을 만든다. 예. `InventoryDatabase.kt`
2. `InventoryDatabase` 클래스를 `RoomDatabase`를 확장하는 `abstract` 클래스로 만든다.
3. 클래스에 `@Database` 주석을 단다. 이 주석에는 `Room`이 데이터베이스를 빌드할 수 있도록 인수가 여러 개 필요하다.
   - entities: 엔티티 클래스
   - version: 데이터베이스 버전 번호. 테이블의 스키마를 변경할 때마다 버전 번호를 높여야 한다.
   - exportSchema: 스키마 버전 기록 백업 여부

   ```kotlin
   @Database(entities = [Item::class], version = 1, exportSchema = false)
   abstract class InventoryDatabase: RoomDatabase() { }
   ```
5. 데이터베이스가 DAO에 관해 알 수 있도록 `ItemDao`를 반환하는 추상 함수를 선언한다.
   ```kotlin
   abstract fun itemDao(): ItemDao
   ```
6. 객체 내 `companion object` 안에 데이터베이스에 관한 비공개 변수를 선언한다.이 변수는 데이터베이스가 만들어지면 데이터베이스 참조를 유지한다. 이를 통해 주어진 시점에 열린 데이터베이스의 단일 인스턴스를 유지할 수 있다.
   이 변수에 `@Volatile` 주석을 단다. `@Volatile` 주석은 휘발성 변수를 의미하며 이 변수의 값은 캐시되지 않으면 모든 읽기와 쓰기는 기본 메모리에서 이뤄진다. 이러한 기능을 사용하여 `Instance` 값을 항상 최신 상태로 유지하고 모든 실행 스레드에서 동일하게 유지할 수 있다.
   ```kotlin
   @Volatile
   private var Instance: InventoryDatabase? = null
   ```
7. `Instance` 아래에 데이터베이스 빌더에 필요한 `Context` 매개변수를 사용하여 `getDatabase()` 메서드를 정의한다. 이 메서드는 `InventoryDatabase` 유형을 반환한다.
   ```kotlin
   fun getDatabase(context: Context)
   ```
   


