---
title: "이미지 리사이징(feat. 썸네일)"
date: 2023-08-16T20:51:10+09:00
draft: false
summary: " "
tags: ["ios", "image"]
---

'토픽' 프로젝트를 하면서 이미지를 저장하고 로드하는 한 화면에 여러 썸네일을 보여줘야 했다.

처음에 썸네일 크기의 이미지를 불러오지 않고 원본 크기 이미지를 썸네일 이미지뷰에 넣어 개발하고 있었다.
그런데 **한 화면에 보여주는 썸네일의 숫자가 많아지자 프로그램이 점점 느려지기 시작했다.**

처음에는 이미지를 불러오는 작업 자체가 오래 걸리는 건가 했는데 그게 아니었다. 


썸네일 크기의 UIImageView에 원본이미지를 사용할때 느려졌다. **UIImageView는 frame size 보다 큰 이미지를 보여주려고 할 때, 이미지의 줄이거나 일부를 자르는 작업을 자동으로 한다.** 그런 이미지 갯수가 많아지니까 자연히 느려진 것. 너무 당연하게 생각했던 작업이라 여기서 시간이 많이 소요될 거라고 생각하지 못했다.

이미지를 리사이징 하는 방법을 알아본 결과 여러가지가 있었는데, 나는 속도가 빠르고 비교적 사용이 간편한 `ImageIO`를 활용한 방법을 참고했다.


```swift
func resizedImage(at url: URL, for size: CGSize) -> UIImage? {
    let options: [CFString: Any] = [
        kCGImageSourceCreateThumbnailFromImageIfAbsent: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceThumbnailMaxPixelSize: max(size.width, size.height)
    ]
    
    guard let imageSource = CGImageSourceCreateWithURL(url as NSURL, nil),
          let image = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options as CFDictionary)
    else {
        return nil
    }
    
    return UIImage(cgImage: image)
}

```


참고: https://nshipster.com/image-resizing/