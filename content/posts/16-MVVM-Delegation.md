---
title: "Delegate 패턴으로 MVVM 구현하기"
date: 2023-08-16T20:45:13+09:00
draft: true
summary: " "
tags: ["ios", "MVVM"]
---
MVVM을 공부하며 대부분 RxSwift와 함께 실습을 했다. 그러다 MVVM, MVVM-C 등 아키텍쳐에 대한 이해가 우선인 것 같아 아키텍쳐에 대해 공부하고 있다.   
이 글에서는 카운터를 만들어보며 MVVM에 대해 배운점을 정리해보고자 한다.

### MVVM
MVC는 View Controller가 너무 비대해지는 단점이 있다. 또한 뷰와 로직 부분이 엮여있어 로직을 테스트가 어려웠다. 이를 해소하기 위해 뷰는 UI, 로직은 View Model로 역할을 명확하게 나누었다.

View
* 화면에 보여주는 UI와 관련된 계층이다. 뷰모델의 데이터를 화면에 보여주는 작업만 하는 곳이다.
* UIViewController, UIView 등

View Model
* View 로부터 이벤트를 받아 데이터를 처리하는 계층이다.
* 뷰모델의 프로퍼티는 크게 Input을 받아서 Output을 업데이트하는 형식이다.
    * Input: 뷰로부터 받은 이벤트
    * Output: 뷰가 UI를 업데이트 하는데 사용하는 데이터

Model
* 데이터 계층이다.
* 여기서는 간단하게 데이터를 나타내는 객체를 의미한다고 보면 될 것 같다.


Rx 에서는 데이터가 변하면 UI에 이벤트가 발송된다. 여기선 Delegate 패턴을 활용해 직접 이벤트를 발생시켜 UI를 업데이트 할 것이다.
각각을 살펴보자.



CounterViewController
```swift
class CounterViewController: UIViewController {
    let viewModel = CounterViewModel()
    ...
    
    lazy var decButton: UIButton = {
        let decButton = UIButton(type: .system)
        decButton.translatesAutoresizingMaskIntoConstraints = false
        decButton.setTitleColor(.blue, for: .normal)
        decButton.setTitle("-", for: .normal)
        decButton.titleLabel?.font = .systemFont(ofSize: 20)
        decButton.addTarget(viewModel, action: #selector(viewModel.decrement), for: .touchUpInside)
        return decButton
    }()
    
    lazy var incButton: UIButton = {
        let incButton = UIButton(type: .system)
        incButton.translatesAutoresizingMaskIntoConstraints = false
        incButton.setTitleColor(.blue, for: .normal)
        incButton.setTitle("+", for: .normal)
        incButton.titleLabel?.font = .systemFont(ofSize: 20)
        incButton.addTarget(viewModel, action: #selector(viewModel.increment), for: .touchUpInside)
        return incButton
    }()
    
    ...
}

```
버튼을 선언할때 addTarget을 보면 viewModel의 increment, decrement로 이벤트를 넘겨준다.
맨 아래 부분에

CounterViewModel
```swift
class CounterViewModel {
    var delegate: CounterViewModelDelegate?

    // MARK: Output
    var number: Number // Model
    
    init(){
        number = Number()
    }
    
    // MARK: Input
    @objc func increment(){
        number.value += 1
        delegate?.updateNumber(number: number.value)
    }
    
    @objc func decrement(){
        number.value -= 1
        delegate?.updateNumber(number: number.value)
    }
    
    func getNumberValue() -> Int {
        return number.value
    }
}

protocol CounterViewModelDelegate {
    func updateNumber(number: Int)
}

```

Input으로 실행되는 이벤트에서 Model에 해당하는 number를 수정하고 있다. 수정한 숫자를 delegate method로 뷰 컨트롤러에 UI 업데이트를 위임하고 있다.


```swift
extension CounterViewController: CounterViewModelDelegate{
    func updateNumber(number: Int) {
        numberLabel.text = String(number)
    }
}
```

Model
```swift
class Number {
    var value: Int = 0
}
```
 
MVVM은 라우팅을 여전히 뷰가 담당하고 있다는 문제가 있다. 이 문제를 해결하기 위해 나온 것이 MVVM-C 혹은 Coordinator 패턴이다. 이름처럼 MVVM에다가 라우팅을 담당하는 Coordinator를 추가했다.
