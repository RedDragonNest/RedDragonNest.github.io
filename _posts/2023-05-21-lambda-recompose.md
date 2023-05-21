---
title: Lambda로 인한 불필요한 Recompose 제거하기
tags: [Android, Jetpack Compose]
style: fill
color: primary
description: Lambda와 Recompose의 관계를 알아보자
---

Jetpack Compose에서 가장 중요한 것은 이러나 저러나 결국 recompose라고 생각한다.   
이를 잘하기 위해서 호이스팅과 remember를 이용하라 라는 글들은 많이들 봤다.   
그러나 lambda와 recompose에 관련된 내용은 많지 않다.   

## 개요
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            LambdaRecomposeTheme {
                LambdaRecomposeScreen()
            }
        }
    }
}

private fun LambdaRecomposeScreen() {
    Column(
        modifier = Modifier
            .fillMaxSize(),
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        var value1 by remember { mutableStateOf("") }
        var value2 by remember { mutableStateOf("") }
        var value3 by remember { mutableStateOf("") }

        TextField(
            value = value1,
            onValueChange = {
                value1 = it
            }
        )

        TextField(
            value = value2,
            onValueChange = {
                value2 = it
            }
        )

        TextField(
            value = value3,
            onValueChange = {
                value3 = it
            }
        )
    }
}
```
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-05-21-lambda-recompose/lambda-recompose_1.png" caption="테스트 환경" %}
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-05-21-lambda-recompose/lambda-recompose_2.png" caption="recompose count 확인" %}

위는 간단하게 세개의 TextField가 각각 값을 받아 저장하는 코드다.   
해당 코드는 ViewModel없이, 즉 복잡한 논리식 없이 View단으로만 처리한 내용이며 크게 문제될 것은 없다.   
여기서 ViewModel을 이용하여 onValueChange를 사용할 경우 어떻게 되는지, 그리고 문제는 어떻게 발생하는지 알아보자.   

```kotlin
@Composable
private fun LambdaRecomposeScreen(
    vm: MainViewModel
) {
    Column(
        modifier = Modifier
            .fillMaxSize(),
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        val value1 by vm.value1.collectAsState()
        val value2 by vm.value2.collectAsState()
        val value3 by vm.value3.collectAsState()

        TextField(
            value = value1,
            onValueChange = {
                vm.onValueChange1(it)
            }
        )

        TextField(
            value = value2,
            onValueChange = {
                vm.onValueChange2(it)
            }
        )

        TextField(
            value = value3,
            onValueChange = {
                vm.onValueChange3(it)
            }
        )
    }
}


class MainViewModel : ViewModel() {
    private val _value1 = MutableStateFlow("")
    val value1 = _value1.asStateFlow()

    private val _value2 = MutableStateFlow("")
    val value2 = _value2.asStateFlow()

    private val _value3 = MutableStateFlow("")
    val value3 = _value3.asStateFlow()

    fun onValueChange1(value: String) {
        _value1.value = value
    }

    fun onValueChange2(value: String) {
        _value2.value = value
    }

    fun onValueChange3(value: String) {
        _value3.value = value
    }
}
```
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-05-21-lambda-recompose/lambda-recompose_3.png" caption="의도치 않은 전체 recompose" %}
ViewModel에서 각각 Flow로 value1 ~ value3으로 선언.   
onValueChange 함수들을 이용하여 값을 저장.   
값이 변경되면 View단에서 해당 Flow를 구독하게 만들었다.   
그리고 Composable에서 onValueChange에 해당하는 함수를 lambda를 통해 사용하고 있다.   
언뜻 보기에는 문제가 없는 코드로 보인다.   
그러나 recompose를 확인하면 그렇지 않다.   
value1에 해당하는 값이 변경되어 첫번째 TextField의 값이 변경되며 recompose하는 것은 당연한 것이다.   
그런데 왜 두번째와 세번째도 같이 recompose되는 것일까?   
이유는 recompose로 인한 변화가 생기면 lambda를 재생성한다는 것에 있다.   
recompose의 문제를 찾기 번거롭고 힘들며, 그게 lambda라고 생각하는 사람은 생각보다 많지 않다.   
해결 방법은 아래와 같다.   

## 해결 1
```kotlin
class MainActivity : ComponentActivity() {
    private val vm: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val onValueChange1: (String) -> Unit = {
            vm.onValueChange1(it)
        }

        val onValueChange2: (String) -> Unit = {
            vm.onValueChange2(it)
        }

        val onValueChange3: (String) -> Unit = {
            vm.onValueChange3(it)
        }

        setContent {
            LambdaRecomposeTheme {
                LambdaRecomposeScreen(
                    vm = vm,
                    onValueChange1 = onValueChange1,
                    onValueChange2 = onValueChange2,
                    onValueChange3 = onValueChange3
                )
            }
        }
    }
}

@Composable
private fun LambdaRecomposeScreen(
    vm: MainViewModel,
    onValueChange1: (String) -> Unit,
    onValueChange2: (String) -> Unit,
    onValueChange3: (String) -> Unit
) {
    val value1 by vm.value1.collectAsState()
    val value2 by vm.value2.collectAsState()
    val value3 by vm.value3.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize(),
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        TextField(
            value = value1,
            onValueChange = onValueChange1
        )

        TextField(
            value = value2,
            onValueChange = onValueChange2
        )

        TextField(
            value = value3,
            onValueChange = onValueChange3
        )
    }
}
```
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-05-21-lambda-recompose/lambda-recompose_4.png" caption="첫번째 해결방법 recompose 확인" %}
첫번째 해결방법은 lambda를 재생성되지 않게 composable 외부로 빼는 것이다.   
일명 호이스팅이라고 하며, 호이스팅을 최상단까지 올려서 주입하는 방식으로 처리한다.   
recompose 표를 보면 확실한 처리방법이라고 볼 수 있다.   

## 해결 2
```kotlin
@Composable
private fun LambdaRecomposeScreen(
    vm: MainViewModel
) {
    val value1 by vm.value1.collectAsState()
    val value2 by vm.value2.collectAsState()
    val value3 by vm.value3.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize(),
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        TextField(
            value = value1,
            onValueChange = remember {
                { vm.onValueChange1(it) }
            }
        )

        TextField(
            value = value2,
            onValueChange = remember {
                { vm.onValueChange2(it) }
            }
        )

        TextField(
            value = value3,
            onValueChange = remember {
                { vm.onValueChange3(it) }
            }
        )
    }
}
```
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-05-21-lambda-recompose/lambda-recompose_5.png" caption="두번째 해결방법 recompose 확인" %}
lambda를 remember로 감싸주는 방법이다.   
변수뿐 아니라 함수도 가능하다.

## 해결 3
```kotlin
@Composable
private fun LambdaRecomposeScreen(
    vm: MainViewModel
) {
    val value1 by vm.value1.collectAsState()
    val value2 by vm.value2.collectAsState()
    val value3 by vm.value3.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize(),
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        TextField(
            value = value1,
            onValueChange = vm::onValueChange1
        )

        TextField(
            value = value2,
            onValueChange = vm::onValueChange2
        )

        TextField(
            value = value3,
            onValueChange = vm::onValueChange3
        )
    }
}
```
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-05-21-lambda-recompose/lambda-recompose_6.png" caption="세번째 해결방법 recompose 확인" %}
마지막으로는 ViewModel의 함수를 직접 참조하는 것이다.   
이렇게 되면 lambda를 따로 생성하지 않고 바로 ViewModel의 내부함수로 바로 접근하게 되니,   
recompose의 복잡한 방식을 크게 이해하지 않고 사용할 수 있다.   

## 그래서 어떤 해결방법으로 써?
해당 부분부터는 개인차다.   
나는 세번째 방식을 메인으로 사용하고 있지만, 다른 방식도 간헐적으로 사용하고 있다.

```kotlin
class MainActivity : ComponentActivity() {
    private val vm: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            LambdaRecomposeTheme {
                LambdaRecomposeScreen(
                    vm = vm,
                    onTestFunc = ::onTestFunc
                )
            }
        }
    }

    private fun onTestFunc() {
        ...
    }
}
```

첫번째 방식을 사용할 때는 activity의 함수를 이용할 때 사용한다.   
내가 개발하고 유지보수 하고 있는 어플리케이션은 Compose로 완전히 마이그레이션하지 않은 상태이다.   
그러다보니 acitivty의 함수들을 이용해야 할 때가 있다.(fragment 관리나 여러 custom helper...)   
해서 위와 같이 사용하여 함수를 이용하고 있다.   

```kotlin
@Composable
private fun LambdaRecomposeScreen(
    vm: MainViewModel
) {
    val value by vm.value.collectAsState()
    var count by remember { mutableStateOf(0) }

    Column(
        modifier = Modifier
            .fillMaxSize(),
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        TextField(
            value = value,
            onValueChange = 
            onValueChange = remember {
                { 
                    count++
                    vm.onValueChange3(it) 
                }
            }
    }
}
```
두번째 방식을 이용할 때는 단순히 논리식만 이용하지 않을때다.   
Composable 내부에서 어떤 작업을 해야할 때 lambda를 remember로 감싸 사용하고 있다.   
lambda와 recompose 사이에서 문제가 생긴 사람들에게 도움이 됐으면 하며,   
세가지 방법을 기억하고 사용하여 좀 더 성능좋은 안드로이드 어플리케이션을 개발했으면 좋겠다.
