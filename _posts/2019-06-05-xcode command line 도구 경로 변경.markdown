---
layout: post
title:  "xcode command line 도구 경로 변경"
date:   2019-06-05 13:07:00 +0900
categories: macOS xcode
---
코코아 어플리케이션을 위한 패키지 관리 도구인 Carthage 를 이용하여 사용할 프레임워크를 설정하는 도중 다음과 같은 오류가 발생하였다. 

```
➜  ProjectName git:(develop) ✗ carthage update
*** Cloning RxSwift
*** Cloning Alamofire
*** Checking out Alamofire at "4.8.1"
*** Checking out RxSwift at "4.4.0"
*** xcodebuild output can be found in /var/folders/xf/8j3d10mj38l2jldz36f5ld800000gn/T/carthage-xcodebuild.JU9AUU.log
*** Downloading RxSwift.framework binary at "Atomic"
***  Skipped installing RxSwift.framework binary due to the error:
	"Incompatible Swift version - framework was built with 4.2.1 (swiftlang-1000.11.42 clang-1000.11.45.1) and the local version is 4.2.1 (swiftlang-1000.0.42 clang-1000.10.45.1)."

    Falling back to building from the source
A shell task (/usr/bin/xcrun xcodebuild -workspace /Users/jookwang/Work/Personal/ProjectTimeCamera/Carthage/Checkouts/Alamofire/Alamofire.xcworkspace CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= CARTHAGE=YES -list) failed with exit code 72:
xcrun: error: unable to find utility "xcodebuild", not a developer tool or in PATH
```

로그 상에서 원인은 다음과 같은 것으로 분석된다.

- Incompatible Swift version, framework는 4.2.1 (swiftlang-1000.11.42 clang-1000.11.45.1)로 빌드되어 있고, 본인의 시스템 환경은 동일한 4.2.1 이지만 상세한 버전이 다르다(swift-1000.0.42 clang-1000.10.45.1). 
- "xcodebuild"란 도구를 찾을 수 없다. 

이를 기반으로 PATH 환경변수를 확인을 했지만 이상한 점을 찾을 수 없었다. 검색을 통해서 xcode-select를 가지고 xcode에서 사용하는 툴에 대한 경로를 확인을 해보라는 답을 얻었다. 그래서, 다음과 같이 경로를 확인해보았다. 

```
➜  ProjectName git:(develop) ✗ xcode-select -p
/Library/Developer/CommandLineTools
```

실행 결과를 보았을 때 xcode에서 설치한 command line 도구에 대한 경로로 지정되어 있었다. 이 때문에 swift, clang 버전과 xcodebuild 툴을 찾을 수 없었던 것으로 보인다. 이를 xcode 경로로 다음과 같이 변경하였다.

```
➜  ProjectName git:(develop) ✗ sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
Password:
➜  ProjectName git:(develop) ✗ xcode-select -p                                                      
/Applications/Xcode.app/Contents/Developer
```

이후 Carthage 명령어를 통해서 프레임워크 업데이트를 진행하니 잘 작동되었다.