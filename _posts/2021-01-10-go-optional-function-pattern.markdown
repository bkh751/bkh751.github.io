---
layout: post
title:  "go functional options pattern"
date:   2020-12-29 01:18:45 +0900
categories: golang
---

go 개발 방법 중 functional options pattern 을 간단한 실습 예제 코드를 통해 소개드립니다.

functional option 은 함수/메서드를 수행할때 가변적인 옵션을 줄 수 있는 방법입니다.

함수/메서드를 작성하고자할때, 아래와 같은 상황일때 유용합니다.

1. default parameter 를 설정하고싶다. 
2. optional한 파라미터 리스트를 가변적으로 설정하고싶다. 

### 구현 방법

1. optional 한 파라미터 리스트 역할을 해주는 `option struct` 를 선언
2. 위의 struct 를 포인터 참조하여 값을 설정해주는 `functional options` 작성
3. 구현하고자 하는 함수 시그니쳐에 가변인자 `functional options` 파라미터를 설정하여 구현


### 간단한 예제 코드

예를들어 외출을 하고싶은 저는 GoOut() 이라는 function 을 수행하고싶습니다. 
GoOut() 을 할때는 muffler, coat 를 입고나가면 좋을거같습니다. 그래서 이 muffler, coat 를 입고나갈지 결정하는 bool 플래그들을 optional 한 파라미터로 가진다고 가정해보겠습니다.

**1. option struct 설정**

그래서 muffler, 와 coat 라는 bool field 를 가지는 `GoOutOption` 라는 struct를 선언해줍니다. 해당 struct 는 이제 구현하고자하는 GoOut() function 의 optional 한 parameter 리스트로써의 역할을 하는 `option struct` 가 됩니다. 
```go
type GoOutOption struct{
  muffler bool
  coat bool
  
  ... // 더 추가 하고 싶은 parameter 들을 field 로 둘 수 있습니다.
}
```

**2. functional options 작성**

GoOutOption 의 muffler 에 접근하여 값을 설정해주는 function 을 리턴해줍니다. 여기서 리턴 되는 `func(option *GoOutOption)`이 `functional option` 이 됩니다. 

coat 의 경우도 마찬가지로 설정이 가능하도록 작성해줍니다.

```go
func WithMuffler(isEnabled bool){
  return func(option *GoOutOption){ 
    option.muffler = isEnabled
  }
}

func WithCoat(isEnabled bool){
  return func(option *GoOutOption){ 
    option.muffler = isEnabled
  }
}
```

**3. 가변인자 functional options 파라미터를 설정하여 구현**

저는 coat 는 기본적으로 입고 나가고 muffler 는 기본적으로 입고 나가지 않는 설정을 default 로 주되 optional 하게 설정하고싶습니다.

그래서 내부적으로 GoOutOption 를 가지고, coat 는 default 값이 true 이고 muffler 는 false 가 되도록 GoOut() 을 구현합니다.

여기서 functional option 인 `func(option *GoOutOption)` 를 가변인자로 설정해주면 파라미터 리스트를 가변적으로 가지는 함수로 만들 수 있게됩니다.

```go
func GoOut(optFuncs ...func(option *GoOutOption)){
  
  //1.에서 선언한 option struct 값을 default 값으로 설정
  option := GoOutOption{ 
    coat : true 
    muffler: false
  }
  
  //option struct 값을 설정해주는 가변인자 functional options 수행
  for i:= range optFuncs{
    optFuncs[i](&option) 
  }

  //GoOut~
}

```

이제 GoOut() 을 수행할때 functional options 을 가변적으로 넘길 수 있습니다. 

```go
// 밖으로 나갈 때 coat 와 Muffler 를 두르고 나간다.
GoOut(WithMuffler(true))

// 밖으로 나갈때 coat 를 입지 않고 나간다.
GoOut(WithCoat(false))

// 밖으로 나갈 때 coat 만 입고 나간다.
GoOut()

```
다른 옵션을 추가하는 부분도 위에 언급드린 1~3 의 과정을 거치면 쉽게 추가할 수 있습니다.

golang option pattern 관련 글
- https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis
- https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
- https://www.sohamkamani.com/golang/options-pattern/

