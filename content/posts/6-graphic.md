---
title: "뷰에 원하는 모양을 그려보자."
date: 2023-01-21T11:33:57+09:00
draft: false
summary: "View, CoreAnimation을 이용해 서큘러 프로그래스 바 그리기"
tags: ["ios", "app"]
---

Quartz 쿼츠는 iOS, Mac 운영체제에서 사용하는 그래픽 엔진이다.   
쿼츠가 실제 그래픽을 뷰에 그릴때 Graphic Context를 가지고 그린다. 컨텍스트엔 색상, 크기 같은 정보와 디바이스에 관련된 정보들이 담겨있다. 여기서 디바이스란 pdf, bitmap, printer, layer 등을 의미한다. 



### 그래픽 컨텍스트를 뷰에 그리기
* ios 앱의 스크린에 뷰를 그리려면 먼저 UIView 객체를 만들고 그리기를 수행하는 `draw(rect:)` 메소드를 구현해야한다.
* `draw(rect:)` 메소드는 뷰가 화면에 보일때와 그 내용이 업데이트가 필요할때 호출된다.
* 구현한 `draw(rect:)` 메소드가 호출되기 전에, 뷰 객체는 자동으로 필요한 환경을 구성한다. 이때 뷰 객체는 현재 환경에 맞는 그래픽 컨텍스트를 생성한다.

아래는 컨텍스트를 이용해 테두리만 있는 원을 그리는 코드이다.
```swift
override func draw(_ rect: CGRect) {
    guard let context = UIGraphicsGetCurrentContext() else { return }
            
    context.beginPath()
    context.setLineWidth(10)
    context.setStrokeColor(UIColor.white.cgColor)
    context.addArc(center: self.center, radius: 30, startAngle: 0, endAngle: 2 * .pi, clockwise: false)
    context.strokePath()
}
```


## 만약 애니메이션을 그리고 싶다면 어떻게 될까?  
뷰의 draw 메소드를 반복적으로 호출해서 UI에 변화를 줄 수 있겠지만 이런 방법은 자원을 많이 사용하는 비싼 방법이다. 왜냐하면 바쁜 메인 스레드에서 동작하기 때문. 그럴땐 애니메이션을 그릴 수 있는 *CoreAnimation* 코어 애니메이션을 사용한다.  

코어 애니메이션은 뷰가 아닌 레이어를 사용해서 그림을 그린다. 레이어는 뷰를 직접 그리지 않고, 현재의 뷰의 인터페이스를 가져와서 비트맵 형태로 관리만 한다. 그럼 이 비트맵 정보를 하드웨어가 그린다. 그렇기 때문에 메인스레드에서 하는 뷰의 그리기 작업보다 훨씬 빠른 것이다.  

레이어를 그리면 실제 레이어의 구조를 그대로 가져와서 Presentation 레이어를 만들고 실제로는 Presentation 레이어를 화면에 보여준다. 따라서 에니메이션의 current value를 얻고 싶으면 presentation 레이어를 참고해야한다.  

실제 모양과 관련된 값을 변형하는 것이 아닌, 그것의 presentation인 레이어의 값을 변형하는 것이 에니메이션의 핵심이다. 따라서 에니메이션이 끝나면 원래 값으로 다시 그리기 때문에 에니메이션이 끝난 후 그 모습을 유지하고 싶으면 에니메이션 시작 후 직접 값을 변형해주어야 한다.

원형프로그래스바를 만들기 위해 뷰의 레이어에 서브레이어를 추가하는 코드이다.  

```swift
var progressBarLayer = CAShapeLayer()

progressBarLayer.strokeColor = progressTintColor
progressBarLayer.fillColor = UIColor.clear.cgColor
progressBarLayer.lineWidth = lineWidth
progressBarLayer.lineCap = .round
progressBarLayer.lineJoin = .round
layer.addSublayer(progressBarLayer)
```

그리고 그림을 그릴 패스를 추가하고 위치를 설정했다.



```swift
override func layoutSubviews() {
    super.layoutSubviews()
    
    let path = UIBezierPath(arcCenter: .zero, radius: radius, startAngle: CGFloat(-90).radian(), endAngle: CGFloat(270).radian(), clockwise: true)
    let center = CGPoint(x: bounds.midX, y: bounds.midY)

    ...
    
    progressBarLayer.path = path.cgPath
    progressBarLayer.position = center
}
```

이렇게 추가해주면 레이어의 정보들을 따라 그림을 그린다. 여기에 에니메이션을 만들어서 추가해주면 `add`하는 즉시 실행된다. 애니메이션하고 싶은 레이어의 속성을 키로 지정하여 객체를 생성한뒤 지속 시간이나 보여줄 값을 지정하는 방식이다.

```swift
let animation = CABasicAnimation(keyPath: "strokeStart")
animation.duration = duration
animation.fromValue = strokeStart
animation.toValue = 1.0
animation.delegate = self
animation.isRemovedOnCompletion = false 
progressBarLayer.add(animation, forKey: "strokeStart")
```


애니메이션이 시작하거나 끝났을때 이벤트를 처리하고 싶다면 CAAnimationDelegate를 사용할 수 있다. 위에 언급한 것처럼 에니메이션이 끝난 후 모습을 유지하기 위해 프로그래스바의 시작 위치 값을 에니메이션이 끝난 후의 값인 1로 변경해주었다.

```swift
extension CircularPrograssBar: CAAnimationDelegate {
    
    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        if flag {
            progressBarLayer.strokeStart = 1
        }
    }
}
```

위에 언급한 대로 presentation 레이어로 에니메이션의 현재 값을 얻어올 수 있다.
```swift
func getPresentationStrokeStart() -> CGFloat? {
    return progressBarLayer.presentation()?.strokeStart
}
```


애니메이션은 실제 동영상을 플레이하는 것과 비슷하게 속도나 시작시간 같은 속성들이 있다. 이것을 조절하면 애니메이션을 일시정지시키거나 다시 재개시킬 수 있다.
```swift
func pause() {
    let pausedTime : CFTimeInterval = self.convertTime(CACurrentMediaTime(), from: nil)
    self.speed = 0.0
    self.timeOffset = pausedTime
}

func resume(){
    let pausedTime = self.timeOffset
    self.speed = 1.0
    self.timeOffset = 0.0
    self.beginTime = 0.0
    let timeSincePause = self.convertTime(CACurrentMediaTime(), from: nil) - pausedTime
    self.beginTime = timeSincePause
}
```


아래는 플레이그라운드에서 간단히 만들어본 예제이다. 

먼저 레이어에 흰색 원 테두리를 그려보자.

```swift
import UIKit
import PlaygroundSupport

let view = UIView()
view.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
view.backgroundColor = .systemMint

let layer = CAShapeLayer()
layer.strokeColor = UIColor.white.cgColor
layer.fillColor = UIColor.clear.cgColor
layer.lineWidth = 10
layer.lineCap = .round
layer.lineJoin = .round

let path = UIBezierPath(arcCenter: .zero, radius: 60, startAngle: 0, endAngle: 2 * .pi, clockwise: true)
let center = CGPoint(x: view.frame.midX, y: view.frame.midY)

layer.path = path.cgPath
layer.position = center
let progressBar = UIView()

progressBar.layer.addSublayer(layer)
view.addSubview(progressBar)

PlaygroundPage.current.liveView = view

```

![원](/image/6-1.png)


그 다음 레이어에 애니메이션을 추가하면 프로그래스바 처럼 움직인다.

```swift
let animation = CABasicAnimation(keyPath: "strokeStart")
animation.duration = 3
animation.fromValue = 0
animation.toValue = 1
animation.repeatCount = .greatestFiniteMagnitude

layer.add(animation, forKey: "animation")
```

![원](/image/6-2.gif)

참고
1. [Core Animation Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514-CH1-SW1)