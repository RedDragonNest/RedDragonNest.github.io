---
title: Flutter 설치하기 (feat. Android Studio / Mac)
tags: [Flutter, Android Studio, Mac, M1]
style: fill
color: info
description: Flutter를 설치해보자
---

해당 포스트는 다음 환경 기준으로 작성되었습니다.

```
OS : Mac OS (M1)
IDE : Android Studio
```


## 1. Android Studio 설치

[#Link](https://developer.android.com/studio)
Flutter는 Android Studio 혹은 Visual Studio에서 손쉽게 개발할 수 있습니다.
해당 포스트는 Android Studio에서 Flutter를 설치하는 방향으로 작성하겠습니다.
링크를 클릭하여 Android Studio를 설치합니다.
{% include elements/figure.html image="https://github.com/RedDragonNest/RedDragonNest.github.io/blob/2683d6b5db71ee783f7a86113409bf85c79e75f4/assets/2022-12-22-install-flutter/install_flutter_1.png" caption="Intel인지 Mac Chip인지 확인하고 설치하자" %}


## 2. Xcode 설치

Xcode는 macOS 전용 IDE입니다.
iOS/macOS용 소프트웨어를 최종 컴파일하려면 Xcode가 필수입니다.
Flutter는 크로스플랫폼을 지향하기 때문에 Xcode가 필요합니다.
App Store에서 Xcode를 검색해서 설치합니다.
{% include elements/figure.html image="https://github.com/RedDragonNest/RedDragonNest.github.io/blob/2683d6b5db71ee783f7a86113409bf85c79e75f4/assets/2022-12-22-install-flutter/install_flutter_2.png" caption="Xcode" %}


## 3. Chrome 설치

Chrome은 현재 가장 대중적인 웹 브라우저입니다.
Flutter는 웹으로도 소프트웨어를 만들 수 있으니 설치합니다.
{% include elements/figure.html image="https://github.com/RedDragonNest/RedDragonNest.github.io/blob/2683d6b5db71ee783f7a86113409bf85c79e75f4/assets/2022-12-22-install-flutter/install_flutter_3.png" caption="Chrome" %}


## 4. Flutter SDK 설치

[#Link](https://docs.flutter.dev/get-started/install/macos)
안드로이드, iOS, 웹. 세 플랫폼의 기초 설치는 끝났습니다.
이제 진짜 Flutter를 설치하는 단계입니다.
Flutter를 개발하기 위해 링크를 클릭하여 SDK를 설치합니다.
Apple M 시리즈라면 Apple Silicon 버전으로 설치합니다.
{% include elements/figure.html image="https://github.com/RedDragonNest/RedDragonNest.github.io/blob/2683d6b5db71ee783f7a86113409bf85c79e75f4/assets/2022-12-22-install-flutter/install_flutter_4.png" caption="Intel인지 Mac Chip인지 확인하고 설치하자" %}


## 5. Rossetta 설치

[#Link](https://github.com/flutter/flutter/wiki/Developing-with-Flutter-on-Apple-Silicon)
Rossetta는 호환을 위해 설치해야 하는 백그라운드 프로그램입니다.
Intel에서 M시리즈로 Mac CPU가 바뀌게 되면서 호환되지 않는 프로그램들이 많아졌습니다.
이를 해결해주는 프로그램입니다.
터미널에 아래 문구를 입력하여 Rossetta를 설치합니다.
> $ sudo softwareupdate --install-rosetta --agree-to-license


## 6. Android Studio Flutter Plugin 설치

Android Studio를 실행한 뒤 Plugin을 설치해야합니다.
> Android Studio -> Preferences -> Plugins -> Marketplace -> flutter 검색
Flutter를 설치하면 자연스럽게 Flutter 개발언어인 Dart도 설치하자고 제안하는데 수락하여 설치합니다.
{% include elements/figure.html image="https://github.com/RedDragonNest/RedDragonNest.github.io/blob/2683d6b5db71ee783f7a86113409bf85c79e75f4/assets/2022-12-22-install-flutter/install_flutter_5.png" caption="Installed를 눌러 설치되어 있는지 확인" %}
{% include elements/figure.html image="https://github.com/RedDragonNest/RedDragonNest.github.io/blob/2683d6b5db71ee783f7a86113409bf85c79e75f4/assets/2022-12-22-install-flutter/install_flutter_6.png" caption="Dart도 같이 설치하자" %}


