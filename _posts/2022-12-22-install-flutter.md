---
title: Flutter 설치하기 (feat. Android Studio / Mac)
tags: [Flutter, Android Studio, Mac, M1]
style: fill
color: info
description: Flutter를 설치해보자
---

해당 포스트는 다음 환경 기준으로 작성되었다.

```
OS : Mac OS (M1)
IDE : Android Studio
```


## 1. Android Studio 설치 [#Link](https://developer.android.com/studio)

Flutter는 Android Studio 혹은 Visual Studio에서 손쉽게 개발할 수 있다.   
해당 포스트는 Android Studio에서 Flutter를 설치하는 방향으로 작성했다.   
링크를 클릭하여 Android Studio를 설치한다.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_1.png" caption="Intel인지 Mac Chip인지 확인하고 설치하자" %}


## 2. Xcode 설치

Xcode는 macOS 전용 IDE다.   
iOS/macOS용 소프트웨어를 최종 컴파일하려면 Xcode가 필수다.   
Flutter는 크로스플랫폼을 지향하기 때문에 Xcode가 필요하다.   
App Store에서 Xcode를 검색해서 설치한다.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_2.png" caption="Xcode" %}


## 3. Chrome 설치

Chrome은 현재 가장 대중적인 웹 브라우저다.   
Flutter는 웹으로도 소프트웨어 결과가 만들어지니 설치한다.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_3.png" caption="Chrome" %}


## 4. Flutter SDK 설치 [#Link](https://docs.flutter.dev/get-started/install/macos)

안드로이드, iOS, 웹. 세 플랫폼의 기초 설치는 끝났다.   
이제 진짜 Flutter를 설치하는 단계다.   
Flutter를 개발하기 위해 링크를 클릭하여 SDK를 설치한다.   
Apple M 시리즈라면 Apple Silicon 버전으로 설치해야한다.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_4.png" caption="Intel인지 Mac Chip인지 확인하고 설치하자" %}


## 5. Rossetta 설치 [#Link](https://github.com/flutter/flutter/wiki/Developing-with-Flutter-on-Apple-Silicon)

Rossetta는 호환을 위해 설치해야 하는 백그라운드 프로그램이다.   
Intel에서 M시리즈로 Mac CPU가 바뀌게 되면서 호환되지 않는 프로그램들이 많아졌다.   
이를 해결해주는 프로그램이다.   
터미널에 아래 문구를 입력하여 Rossetta를 설치하자.   

> sudo softwareupdate --install-rosetta --agree-to-license


## 6. Android Studio Flutter Plugin 설치

Android Studio를 실행한 뒤 Plugin을 설치해야한다.   

> Android Studio -> Preferences -> Plugins -> Marketplace -> flutter 검색

Flutter를 설치하면 자연스럽게 Flutter 개발언어인 Dart도 설치하자고 제안하는데 수락하여 설치한다.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_5.png" caption="Installed를 눌러 설치되어 있는지 확인" %}

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_6.png" caption="Dart도 같이 설치하자" %}


## 7. PATH 설정

환경설정을 등록하기 위해서는 먼저 4번에서 설치한 Flutter SDK를 압축해제한 폴더 위치가 필요하다.   
터미널을 실행한 후 위치를 확인한다.   

> ls 입력하여 flutter 폴더 확인 -> pwd 입력하여 경로 확인

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_7.png" caption="pwd 결과를 복사해놓자" %}

위치를 확인했다면 다시 터미널을 실행한다.   
그리고 다음과 같이 입력하면 환경설정을 편집할 수 있다.   

> vim ~/.zshrc

환경설정 파일 안에 다음과 같이 입력해준다. 입력 커서가 나오지 않고 아무글도 써지지 않는다면 `i`를 눌러 INSERT 모드인 상태로 변경해준다.   

```
export PATH=/opt/homebrew/bin:$PATH
export PATH=/Users/dragon/Documents/flutter/bin:$PATH  <- [pwd결과]/flutter/bin:$PATH 입력
```

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_8.png" caption="두번째 라인을 확실히 변경하자" %}

두번째 라인에 해당하는 공간에 pwd결과를 넣어준다. 그리고 `esc`를 눌러 INSERT 모드에서 벗어난 뒤 `:wq`를 입력하여 저장한다.   
환경설정 파일을 편집하고 저장하였다면 다음의 명령어를 입력하여 리프레시 해준다.   

> source ~/.zshrc   


***   

## 동작확인 및 에러 대응

터미널에서 `flutter doctor`를 입력하면 Flutter를 사용할 수 있는 환경인지 알아낼 수 있다.   
만약 명령어가 동작하지 않으면 대체로 항목 7번 환경설정이 잘못되있을 것이다.   
다음은 에러들과 각 에러에 대한 대응이다.   

+ cmdline-tools component is missing

  `Android Studio SDK Manager → Appearance & Behavior → System Settings → Android SDK → SDK Tools → Android SDK Command-line Tools`

  위의 경로에 해당하는 부분을 체크한 후 Apply하여 설정해주자.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_9.png" %}

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_10.png" caption="두번째 라인을 확실히 변경하자" %}

+ Android license status unknown

  터미널에 다음을 입력하자.

  `flutter doctor --android-licenses`

  그러면 이것저것 허락해달라고 하는데, 어차피 우리는 y만 눌러야 한다. :)   

+ CocoaPods not installed

  터미널에 다음을 입력하자.

  `sudo gem install cocoapods`

+ Xcode installation is incomplete; a full installation is necessary for iOS development

  Xcode를 설치했음에도 해당 에러가 발생한다면 터미널에 다음을 입력하자.

  `sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer`

+ Xcode end user license agreement not signed

  XCode를 실행하여 라이센스를 동의하자.

+ Unable to find bundled Java version

  Android Studio가 업데이트 되면서 생기는 버그이다. 터미널에 다음을 입력하자.

  `cp -r /Applications/Android\ Studio.app/Contents/jbr /Applications/Android\ Studio.app/Contents/jre`

+ Error: To set up CocoaPods for ARM macOS, run: sudo gem uninstall ffi && sudo gem install ffi -- --enable-libffi-alloc

  Android Studio에서 iOS 에뮬레이터로 실행하려 할 때 오류가 뜬다면, 터미널에 다음을 입력하자.

> sudo gem install ffi -- --enable-libffi-alloc

  + Flutter로 만든 어플리케이션을 실행하려 했으나 디바이스와 Run버튼이 비활성화 되어 있을 경우

  `Android Studio → Preferences → Languages & Frameworks → Flutter → Flutter SDK path`

  위로 들어가서 4번에 해당하는 경로로 설정해주자.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_11.png" caption="4번에 해당하는 경로로 설정" %}

+ SDK 경로를 잘 설정했으나, 디바이스만 잡아지고 실행버튼이 활성화되지 않을 경우

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_12.png" %}

  이를 해결하기 위해서는 실행하고자 하는 '-.dart' (아마 runApp 함수가 들어있는) 파일에서 우클릭 후 Run '-.dart'를 클릭하면 실행되면서 자동으로 Run버튼도 활성화된다.   

{% include elements/figure.html image="https://raw.githubusercontent.com/RedDragonNest/RedDragonNest.github.io/main/assets/2022-12-22-install-flutter/install_flutter_13.png" %}


