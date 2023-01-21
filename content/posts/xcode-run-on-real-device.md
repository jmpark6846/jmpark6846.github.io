Unsupported os version 
xcode에서 실제 기기를 usb에 연결해서 실행하려고 할때 발생한다.
xcode가 지원하는 ios버전에 기기의 ios버전이 포함되지 않을 경우 발생한다.
xcode의 DeviceSupport에 해당 iOS버전의 device support file을 추가한다.

https://github.com/iGhibli/iOS-DeviceSupport/tree/master/DeviceSupport

만약 본인이 원하는 버전의 device support file이 없는 경우 이전 버전을 받아 압축을 푼 후 폴더명을 해당 버전으로 변경해도 된다.

예)
찾는 버전이 `16.2` 인데 `16.1` 밖에 없는 경우 `16.1`을 받아 압축을 푼 후 폴더명을 `16.2`로 변경


압축풀 주소 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/




Failed to prepare device for development

DeviceSupport file을 추가하여 Unsupported os version 이슈를 해결했으나 기기에서 실행해보면 다음과 같은 메세지를 보여주며 작동하지 않는다.

`
Failed to prepare device for development.
This operation can fail if the version of the OS on the device is incompatible with the installed version of Xcode. You may also need to restart your mac and device in order to correctly detect compatibility.
`

나의 경우 iOS16 에 도입된 '개발자 모드'를 활성화하지 않아서 생긴 문제.
그런데 내 아이폰(iPhone 11, iOS 16.2)만 그런지 모르겠으나 공식문서에 나온 것 처럼 '설정 - 개인정보 보호 및 보안'을 들어가봐도 '개발자 모드'가 없었다.


구글링해본 결과
맥북에 'iMyFone AnyTo' 라는 프로그램을 설치해서 연결하면 개발자모드 메뉴가 보여서 활성화했다.
해당 프로그램은 GPS 조작 프로그램인데 크게 문제가 되어 보이진 않았다.
원래는 휴대폰과 xcode를 연결하면 보인다고 한다.

