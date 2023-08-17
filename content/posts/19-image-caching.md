---
title: "NSCache를 이용해 이미지 캐싱하기"
date: 2023-08-17T11:07:50+09:00
draft: false
summary: " "
tags: ["ios", "image", "NSCache"]
---

이번엔 지난 글에 이어 파일로 불러온 이미지를 캐싱해서 사용하는 방법을 알아보자.

URL 위치의 이미지 파일을 불러와 사용한다면 `[URL: UIImage]` 형태의 딕셔너리를 사용할 수 있다. 이 방법을 사용하다보면 이미지 크기가 크거나 갯수가 많아지면 사용하는 메모리가 많아져서 성능에 문제가 생긴다. 메모리 문제를 해결하기 위해 나온 것이 `NSCache`이다.


`NSCache`는 일시적인 데이터를 담는 데 사용하는 데이터 구조이다. 사용방법은 딕셔너리와 거의 유사하며 한 가지 큰 차이점이 있다면 앞서 봤던 메모리 문제를 해결해준다는 점이다. 시스템의 메모리가 부족할때 자동으로 NSCache의 항목을 제거하여 메모리를 회수한다. 

캐시에 저장되는 데이터의 크기나 갯수 cost를 정할 수 있다. 이 cost를 넘어가면 자동으로 항목을 제거하여 메모리를 회수한다. 데이터가 제거되었다면 다시 불러오는 것은 개발자의 몫이다.

`NSCache`를 사용하여 이미지를 캐싱하는 `ImageCache` 를 만들어보자.

```swift
class ImageCache {
    public static let shared = ImageCache()
    private let cache = NSCache<NSURL, UIImage>()

    ...
}
```

직접적으로 캐시에 접근해서 데이터를 가져오는 메소드를 작성했다. 파라메터로 받은 주소에 이미지를 가져와 리턴한다. 해당 주소로 캐싱하지 않았다면 `nil`을 리턴한다.

```swift
public final func image(url: NSURL) -> UIImage? {
    return cache.object(forKey: url)
}
```

이번에는 해당 주소에 이미지가 있으면 이미지를 리턴하고 없으면 파일에서 읽어오는 메소드를 추가해보자. 

```swift
final func getImage(url: NSURL) -> UIImage? {

    if let cachedImage = image(url: url) {
        return cachedImage
    }
    
    if let image = ImageManager.shared.loadImage(url: url.absoluteURL!) {
        cache.setObject(image, forKey: url)
        return image
    }
    
    return nil
}
```

NSCache를 사용하기 위해서는 URL이 아닌 NSURL을 사용해야 한다. URL을 NSURL로 변환하는 메소드를 만들어 사용하였다. (`ImageManager`는 이전 글 참고)

```swift
func nsurl(filename: String) -> NSURL? {
    guard let url = ImageManager.shared.url(filename: filename) else { return nil }
    return NSURL(fileURLWithPath: url.path)
}
```

아래는 최종 코드이다.

```swift

class ImageCache {
    public static let shared = ImageCache()
    private let cache = NSCache<NSURL, UIImage>()
    
    public final func image(url: NSURL) -> UIImage? {
        return cache.object(forKey: url)
    }
    
    final func getImage(url: NSURL) -> UIImage? {

        if let cachedImage = image(url: url) {
            return cachedImage
        }
        
        if let image = ImageManager.shared.loadImage(url: url.absoluteURL!) {
            cache.setObject(image, forKey: url)
            return image
        }
        
        return nil
    }
}
```