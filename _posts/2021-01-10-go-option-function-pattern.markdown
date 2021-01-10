---
layout: post
title:  "go options function pattern"
date:   2020-12-29 01:18:45 +0900
categories: jekyll update
---

요즘들어 자주쓰는 go 개발 패턴이라하면 option function 패턴인데 이걸 굳이 사용하는 이유는 몇가지가 있다. 

1. 개발하다 보면 함수 내부에 default parameter 로 설정하고 싶은 값들이 있을 수 있는데 go 는 이걸 지원하지 않는다. 
2. 기존 코드에서 추가적인 개발을 할때 method overloading 을 통해 함수 시그니쳐에 변화를 줘서 새로운 메소드를 구현하고싶을 때도 있지만 go는 이를 지원하지 않는다. 

그래서 결론적으로 default parameter를 설정하거나 method 시그니쳐 변경을 자유롭게 하고싶다면 option pattern 을 쓰면 가능하다.

### 구현 방법

option pattern 은 간단하다. 

option 을 설정해주는 function 을 구현하면된다. 통상적으로 메소드 명을 어떻게 하는지는 자유지만 보통 with prefix 를 써서 작성해주면된다. 예를들어 밖에 옷을 입고 나갈때 목도리를 두르거나 모자를 쓴다던지 뭐 이런건 다 옵션이니 이를  pseudo code 로 작성하면

### Pseudo code 

```go
type GoOutOptions struct{
  muffler bool
  hat bool
}


func WithMuffler(){
  return func(options *Options){ // 반드시 포인터 옵션을 받는다 그래야 해당 함수가 옵션 값을 변조할 수 있으니까.
    options.muffler = true
  }
}

func WithHat(){
  return func(options *Options){
    options.hat = true
  }
}


func goOut(opts ...options){
  goOutDefault := GoOutOptions{ //이런식으로 default 값 설정이 가능하다
    muffler : false
    hat : false
    coat : true 
    shirts : true
    pants : true
  }

  for i:= range opts{
    opts[i](&goOutDefault) //옵션들을 적용해준다.
  }

  ...
  
}

```

이제 goOut() 을 수행하면된다. 

```go
// 밖으로 나갈 때 모자도 쓰고간다
goOut(WithHat())

// 밖으로 나갈 때 모자도 쓰고 목도리도 한다.
goOut(
  WithHat(),
  WithMuffler(),
)

```


다음 포스트 예정 

reflect 
