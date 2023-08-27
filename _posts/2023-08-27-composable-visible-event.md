---
title: Composable이 화면에 보여질 때 이벤트 받기
tags: [Android, Jetpack Compose]
style: fill
color: success
description: 두가지 버전으로 준비
---

화면에 View가 보여질 때 이벤트를 받아야하는 경우가 있다.   
다른 View에 변화를 준다던가, 기업에서 사용하기 위해 로그를 쌓는다던가.   
Composable에서는 어떻게 이벤트를 받을 수 있는지 살펴보았다.   

## 코드
```kotlin
fun Modifier.isShown(
    isShown: () -> Unit
): Modifier = composed {
    val view = LocalView.current
    var isVisibleState: Boolean? by remember { mutableStateOf(null) }

    if (isVisibleState == true) {
        LaunchedEffect(isVisibleState) {
            isShown()
        }
    }

    onGloballyPositioned { coordinates ->
        isVisibleState = coordinates.isVisibleCenterCalc(view)
    }
}

/**
 * 화면에 composable이 보이면 true 반환
 */
fun LayoutCoordinates.isVisibleCalc(
    view: View
): Boolean {
    if (!isAttached) return false

    val globalRootRect = android.graphics.Rect()
    if (!view.getGlobalVisibleRect(globalRootRect)) {
        return false
    }

    val composableView = boundsInWindow()
    return composableView.top >= globalRootRect.top &&
            composableView.left >= globalRootRect.left &&
            composableView.right <= globalRootRect.right &&
            composableView.bottom <= globalRootRect.bottom
}

/**
 * 화면에 composable의 중앙점이 보이면 true 반환
 */
fun LayoutCoordinates.isVisibleCenterCalc(
    view: View
): Boolean {
    if (!isAttached) return false

    val globalRootRect = android.graphics.Rect()
    if (!view.getGlobalVisibleRect(globalRootRect)) {
        return false
    }

    val composableView = boundsInWindow()

    val heightHalf = size.center.y
    val widthHalf = size.center.x

    return widthHalf < composableView.width &&
            heightHalf < composableView.height
}
```

핵심은 간단하다.   
composable의 그려진 값의 크기를 알고, 부모의 크기를 알아내어 자신만의 계산식에 따라 결과를 return 해주면 된다.   
나는 두가지를 만들어보았는데   
isVisibleCalc 함수는 화면에 composable이 보이는 순간 이벤트를 발생시키고   
isVisibleCenterCalc 함수는 화면에 composable의 중앙이 보이면 이벤트를 발생시켰다.   

## 사용
```kotlin
Box(
    modifier = Modifier
        .size(100.dp)
        .background(Color(0xFFFF9800))
        .isShown {
            logs.add("on show : row : $row / column : $column")
        },
    contentAlignment = Alignment.Center
) {
    Text(
        text = "row : $row / column : $column",
        color = Color.White
    )
}
```
Modifier에 정의해놓은 isShown을 사용하여 이벤트를 받을 수 있다.   

## 결과
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-08-27-composable-visible-event/composable-visible-event-visible-calc.mp4" caption="화면에 보이자마자 이벤트 발생" %}
{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2023-08-27-composable-visible-event/composable-visible-event-visible-center-calc.mp4" caption="화면에 중앙값이 보이면 이벤트 발생" %}
