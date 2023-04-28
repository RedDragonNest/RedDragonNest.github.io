---
title: DataStore 마이그레이션
tags: [Android, Clean Architecture, SharedPreference, DataStore, Hilt]
style: fill
color: danger
description: SharedPreference를 DataStore로 마이그레이션 해보자 (feat. Hilt)
---

## 1. DataStore?

안드로이드 디바이스에서 내부 저장소를 이용하는 방법은 몇가지 있다.   
그 중 가장 쉽게 이용할 수 있는 기능이 바로 SharedPreference다.   
단순히 Key와 Value로 이뤄져있어 쉽게 이해 가능하고 사용 가능하다.   
그러나 몇년전부터 SharedPreference를 대체하려는 구글의 움직임이 보였다.   
구글이 한다고 하고 버려진 프로젝트도 많기 때문에, 바로 적용하지 않았지만 이제는 해도 될 것 같고 실험을 통해 안정적인 것이 확인이 되었다.   
대체제는 바로 DataStore다.   
DataStore는 무엇이고 어떻게 쓰며 SharedPreference에서 마이그레이션은 어떤 방법으로 하는지 알아보려 한다.   

DataStore는 Preferences DataStore 및 Proto DataStore로 나뉘는데,   
SharedPreference와 유사한 것이 Preferences DataStore이기 때문에 해당 방법을 알아보기로 한다.   

## 2. 정의

```kotlin
private val Context.dataStore by preferencesDataStore(
    name = DATA_STORE_NAME,
    produceMigrations = { context ->
        listOf(
            SharedPreferencesMigration(context, SHARED_PREFERENCE_NAME, prefsMigrationList)
        )
    })
```

정의는 name만 정의해주면 되며 필요에 따라 produceMigrations를 같이 파라미터로 넘겨주면 된다.   
마이그레이션은 list로 받을 수 있게 되어있다.   
SharedPreference에서 값을 Key단위로 마이그레이션 할 수 있고, 여러 SharedPreference에서 옮길 수 있다.   

단 여기서 주의할 것은 마이그레이션을 한 Key들은 SharedPreference에서 다시 쓰일일이 없어야 한다.   
그렇지 않으면 마이그레이션과 SharedPreference에 다시 생성된 값이 다시 옮기기 어려워지며, 개발자에게도 혼란을 야기하기 때문이다.   

```kotlin
private object PreferencesKeys {
    val PREFERENCE_VALUE_1 = booleanPreferencesKey("SHARED_PREFERENCE_VALUE_1")
    val PREFERENCE_VALUE_2 = stringPreferencesKey("SHARED_PREFERENCE_VALUE_2")
}

private val prefsMigrationList = setOf(
    PreferencesKeys.PREFERENCE_VALUE_1.name,
    PreferencesKeys.PREFERENCE_VALUE_2.name
)
```

SharedPreference에서 마이그레이션하고 싶은 Key들을 변수 prefsMigrationList에 정의해주면 된다.   
마이그레이션을 원하는 Key들만 할 수 있기 때문에 프로젝트에 해당하는 모든 SharedPreference들을 한번에 옮길 필요가 없어 위험부담도 적다.   
보통 마이그레이션이 매우 불편한데, 이는 마이그레이션을 적극적으로 하기 위한 좋은 장점으로 다가왔다.   

PreferencesKeys.PREFERENCE_VALUE_1와 PREFERENCE_VALUE_2의 정의를 보자.   
각각 booleanPreferencesKey, stringPreferencesKey이다. 이를 통해 DataStore에 어떤 형태로 저장하고 읽을지 정의할 수 있다.   

## 3. 읽기

```kotlin
val prefValue1: Flow<Boolean> = 
	context.dataStore.data
	    .catch {
	    	emit(emptyPreferences())
	    }.map { preferences ->
	        preferences[PREFERENCE_VALUE_1]
	    }
}
```

SharedPreference와 달리 DataStore에서는 값을 읽기 위해 Flow로 값을 return 받아야 한다. DataStore의 주요 이점 중 하나인 비동기이다.   
그리고 혹시 모를 불안전한 접근을 위해 catch하여 올바른 빈값과 프로젝트의 방향에 맞게 error처리를 해준다.   

## 4. 쓰기

```kotlin
context.dataStore.edit { preferences ->
    preferences[PREFERENCE_VALUE_1]
}
```

읽고 쓰기가 비동기이다보니 아무래도 쓰는 쪽은 오히려 간단하다. PreferencesKey의 타입에 맞게만 값을 넣어주면 된다.   

이렇게 DataStore의 정의, 읽기, 쓰기를 알아봤다.   
그러나 실전에서 SharedPreference에서 마이그레이션을 했을 경우는 어떤지를 살펴보아야 한다.   
특히 읽기에서는 비동기로 동작하다보니 기존 SharedPreference의 동기와 다르다보니 기존 논리식을 건드리지 않으려면 처리를 다른 방식으로 해줘야한다.   

```kotlin
private suspend fun getPrefValue1(): Boolean =
	context.dataStore.data
	    .catch {
	        emit(emptyPreferences())
	    }.map { preferences ->
	        preferences[PREFERENCE_VALUE_1]
	    }.firstOrNull() ?: false
}
```

내가 했던 방식은 위와 같다.   
결과를 받는 부분을 Flow로 리턴 후 비동기에서 동기로 바꾸기 위해 firstOrNull을 사용하여 초기화까지 마친다.   
해당 방법을 사용하면 기존 논리식을 크게 건드리지 않고 안전하게 SharedPreference를 사용할 수 있다.   

## 5. 도입

이후는 실제 프로젝트에 어떻게 도입했는지, 그리고 도입한 일부 코드와 예제 코드들이다.   

```kotlin
DataStoreExtension.kt

suspend fun FlowCollector<Preferences>.recoverOrThrow(throwable: Throwable) {
    emit(emptyPreferences())
}

suspend fun DataStore<Preferences>.clear() {
    edit { preferences ->
        preferences.clear()
    }
}

suspend fun DataStore<Preferences>.edit(key: Preferences.Key<Boolean>, value: Boolean) {
    edit { preferences ->
        preferences[key] = value
    }
}

fun Flow<Preferences>.getValue(key: Preferences.Key<Boolean>) =
    catch {
        recoverOrThrow(it)
    }.map { preferences ->
        preferences[key]
    }
```

먼저 DataStore를 간단히 이용하기 위해 Extension을 정의하였다.   
일반적으로 catch한 error들의 처리 방법들이 다르지 않기 때문에 한곳에서 관리하기 쉽게 recoverOrThrow 함수로 정의하였다.   
edit과 getValue함수도 마찬가지다. 해당 행동들은 보통 다르지 않기 때문에 Extension으로 만들어놓으면 간단히 사용할 수 있다.   

```kotlin
ExamplePrefs.kt

interface ExamplePrefs {
    suspend fun getPrefValue1(): Boolean
    suspend fun putPrefValue1(value: Boolean)
    suspend fun getPrefValue2(): String
    suspend fun putPrefValue2(value: String)
}
```

```kotlin
Di.kt

@Module
@InstallIn(SingletonComponent::class)
abstract class AbstractExamplePrefs {
    @Binds
    @Singleton
    abstract fun bindExamplePrefs(
        impl: ExamplePrefsImpl
    ): ExamplePrefs
}

```

나는 의존성 주입을 위한 Hilt를 사용한다. 그러기 위한 interface와 binds다.   

```kotlin
ExamplePrefsImpl.kt

class ExamplePrefsImpl @Inject constructor(
    @ApplicationContext private val context: Context
) : ExamplePrefs {
    
    private val Context.dataStore by preferencesDataStore(
        name = DATA_STORE_NAME,
        produceMigrations = { context ->
            listOf(
                SharedPreferencesMigration(context, SHARED_PREFERENCE_NAME, prefsMigrationList)
            )
        })

    override suspend fun getPrefValue1() =
        context.dataStore.data.getValue(PreferencesKeys.PREFERENCE_VALUE_1).firstOrNull() ?: false

    override suspend fun putPrefValue1(value: Boolean) {
        context.dataStore.edit(PreferencesKeys.PREFERENCE_VALUE_1, value)
    }

    override suspend fun getPrefValue2() =
        context.dataStore.data.getValue(PreferencesKeys.PREFERENCE_VALUE_2).firstOrNull() ?: ""

    override suspend fun putPrefValue2(value: String) {
        context.dataStore.edit(PreferencesKeys.PREFERENCE_VALUE_2, value)
    }

    private object PreferencesKeys {
        val PREFERENCE_VALUE_1 = booleanPreferencesKey("SHARED_PREFERENCE_VALUE_1")
        val PREFERENCE_VALUE_2 = stringPreferencesKey("SHARED_PREFERENCE_VALUE_2")
    }

    private val prefsMigrationList = setOf(
        PreferencesKeys.PREFERENCE_VALUE_1.name,
        PreferencesKeys.PREFERENCE_VALUE_2.name
    )
}
```

## 마무리

이렇게 구현체까지 알아보았다.   
DataStore로 마이그레이션하는 과정 자체는 복잡하지 않다.   
해당 예제는 의존성 주입을 하기 위한 작업들이 들어가 있기 때문에 코드도 많아지고 귀찮아진 것이다.   
꼭 이렇게 할 필요는 없다.   
내가 이렇게 한 이유는 각 모듈간 의존 혹은 클린 아키텍처를 최대한 이용하고, 테스트 코드를 적용할 수 있게 하기 위함이다.   
이렇게 적용한 내용은 View에서 사용하던 ViewModel에서 사용하던 상관 없이 각자 아키텍처 패턴에 맞게 사용하기 바란다.   
나는 MVVM 패턴에서 ViewModel에 사용 중이다.   
