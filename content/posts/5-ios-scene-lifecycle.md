---
title: "앱의 라이프사이클(번역)"
summary: "앱의 라이프사이클과 관련된 문서 번역 및 정리했다."
date: 2023-01-20T22:23:15+09:00
draft: false
tags: ["ios", "app"]
---


### 앱의 상태
* foreground: 사용자와 상호작용. CPU 같은 시스템 자원들에 대한 권한이 더 높다.
* background: 아무일도 안하거나 적은 일을 해야한다.


앱의 상태가 변하면 UIKit은 delegate 객체에게 이벤트를 전달하여 알맞은 메소드가 실행되도록 한다.  
iOS13 이상 부터는 Scene기반으로 동작하여 UISceneDelegate 을 사용하고 iOS12 이하는 UIApplicationDelegate를 사용한다.  


### Scene 
앱의 UI 인스턴스를 말한다. 앱은 하나의 프로세스를 가지지만 여러개의 scene을 가질 수 있다. 따라서 각각의 scene마다 다른 상태를 가질 수 있고, 상태가 변할때마다 이벤트가 scene delegate 로 전달되어 처리될 수 있다.



## 작동 흐름
1. 사용자나 시스템이 앱의 새로운 scene을 요청하면 UIKit은 unattached state 씬을 생성한다. 
2. 사용자가 요청한 씬은 foreground로 이동해 화면에 보여진다.
3. 시스템이 요청한 씬은 보통 background로 이동해 이벤트를 처리한다.
4. 만약 사용자가 앱의 UI를 dismiss할 경우, UIKit은 해당 씬을 background로 옮기고 결국엔 suspended state로 이동시킨다.
5. UIKit은 자원을 회수하기 위해, background나 suspended 씬을 언제든 disconnect 시켜 씬을 unattached state로 돌려놓는다.



### 앱의 상태들
* Active: 앱이 사용되고 있는 상태
* Background: 앱이 화면에는 없지만 여전히 코드를 실행시키고 있음
* Suspended: 여전히 메모리에는 있지만 코드를 실행하진 않음 



![ios-app-lifecyle](/image/ios-app-lifecyle.png )

## Foreground 진입


### sceneWillEnterForeground (unattched, background -> inactive)
데이터를 디스크에서 불러오거나 네트워크에서 fetch 해오자.


### sceneDidBecomeActive (inactive -> active)
UI구성하고 초기 작업들 실행하자.
* 앱의 윈도우를 보여주기
* 현재 보이는 뷰 컨트롤러 변경하기
* 데이터를 업데이트 하고 뷰와 컨트롤의 상태 변경하기
* 일시 정지한 게임을 다시 시작하기 위해 컨트롤을 display하기
* task 를 실행하기 위해 필요한 디스패치큐를 시작하거나 재개하기
* data source 객체 업데이트하기
* 타이머 실행하기


  
### viewWillAppear 
SceneDelegate 메소드가 아닌 view controller 의 이벤트 메소드.
Activation 메소드가 끝나면 UIKit은 화면에 윈도우들을 보여준다. 또한 모든 뷰컨트롤러에게 뷰가 곧 나타날 것(appear)임을 알려준다.
* UI 에니메이션 시작하기
* 미디어 재생하기 (자동재생일 경우)
* 게임의 그래픽 보여주기 

이때는 다른 뷰 컨트롤러를 보여주거나 UI에 큰 변화를 주지 말자. 뷰 컨트롤러가 화면에 나타날 즈음엔 인터페이스가 준비되어 있어야 한다.



## Background 진입
앱은 여러가지 이유로 백그라운드 상태가 된다.  
* 사용자가 foreground 앱을 종료시킬때 UIKit은 앱을 background로 이동시켰다가 나중에 suspend 시킨다.
* 시스템은 앱 시작 시 바로 background state로 시작할수도 있다.
* suspend된 앱을 백그라운드 상태로 만들거나 중요한 작업들을 하도록 백그라운드 상태로 둘수도 있다.

만약 앱이 백그라운드 상태가 되면, 가능한 적게 혹은 아무런 일도 안하도록 하는것이 좋으며 가능한 빨리 작업을 완료해야한다.



### sceneWillResignActive (active -> inactive)
* 사용자가 foreground 앱을 종료시켰을 때, 시스템은 앱을 비활성화 시킨 후 백그라운드로 진입시킨다.
* 시스템은 앱을 잠깐 인터럽트 시켜야할때 비활성화 시킨다. 예를들면 시스템 얼럿을 보여줘야할때. 이런 경우 사용자가 시스템 패널을 dismiss하면 앱은 재활성화 된다.


비활성화가 될때 사용자의 데이터를 보존하거나 중요한 작업들을 잠시 멈추자. 
* 사용자의 데이터를 디스크에 저장하거나 열려진 파일들을 닫기 
* 디스패치큐나 작업큐를 서스팬드하기
* 새로운 작업을 스케쥴 하지 말것
* 모든 타이머들을 멈출 것
* 게임플레이를 멈출 것



### sceneDidEnterBackground: (inactive -> background)

* 앱이 백그라운드로 전환되면 앱이 사용하던 메모리나 공유자원들을 release 하자.
* foreground 에서 백그라운드로 전환하는 앱의 경우 메모리를 free 하는 것이 특히 중요하다. foreground 앱은 메모리나 다른 시스템 자원에 대한 우선순위를 갖기 때문에 시스템은 자원을 얻어야 한다면 백그라운드 앱들을 terminate 시키기 때문이다. 만약 앱이 foreground에 있지 않더라도 항상 가장 적은 자원을 사용하도록 확인 할것.

백그라운드로의 전환시 다음 작업들을 수행하자.
* 파일로 읽어들인 이미지나 미디어를 discard하기
* 디스크로 읽어들였거나 다시 만들 수 있는 in-memory 객체들을 discard하기
* 카메라를 포함한 하드웨어 자원에 대한 접근을 release하기
* 앱의 UI에 패스워드 같은 민감한 정보를 숨기기
* 얼럿 같은 일시적인 UI들을 dismiss하기
* shared system database에 대한 연결을 끊기

앱이 어떠한 시스템의 공유 자원들도 가지고 있지 않도록 하자. 만약 백그라운드에 진입했는데도 계속해서 카메라나 shared system database 들에 접근한다면 시스템은 자원을 얻기위해 앱을 terminate 시킬 것이다.


## 앱 스냅샷 준비하기

앱이 백그라운드로 진입하고 delegate 메소드가 리턴하면, UIKit은 앱의 현재 UI의 스냅샷을 찍는다. 이 이미지는 앱 스위처에서 사용된다. 또한 앱을 다시 foreground로 이동시킬때 잠시동안 보여준다.  

이때 사용자의  민감한 정보들을 담지 않아야 한다. 만약 그런 정보들이 UI에 있다면 백그라운드로 진입할때 지워주자(스냅샷에 찍히니까).   

스냅샷은 사용자에게 잘 보여야 하므로 앱의 컨텐츠들을 가리는 얼럿이나 일시적인 인터페이스들, 시스템 뷰 컨트롤러들을 백그라운드 진입 시 제거하자.  



## 백그라운드에서 이벤트 처리하기

앱은 백그라운드에 진입하면 실행시간을 더 할당 받지 못한다. 하지만 몇가지 경우에는 실행시간을 할당한다.

* AirPlay를 사용한 오디오 커뮤니케이션, PIP 비디오
* Location 관련 서비스
* VoIP
* 외부 악세서리를 사용한 커뮤니케이션
* 서버로부터의 정기적인 업데이트
* APNs 



### 참고

1. [Managing your app’s life cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)


2. [Preparing your UI to run in the foreground](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_foreground)

3. [Preparing your UI to run in the background](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)



