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

다음은 Entity 클래스를 만드는 과정이다.

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


