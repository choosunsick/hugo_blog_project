---
title: "Rust 트레이트(Trait)"
date: 2021-07-30T17:51:01+09:00
draft: False
tags: ["이산수학","Python"]
categories: ["이산수학","Python"]
---

## 개념 소개

Rust에는 Trait이라는 기능이 있습니다. 이 기능은 러스트의 컴파일러에게 특정 타입이 어떤 기능을 수행할 수 있고 그 기능을 다른 타입들과 공유할 수 있는지를 알려주는 역할을 합니다. 예를 들면 만약 plus 라는 더하기 함수가 문자열 타입과 숫자 타입 모두에 적용 되게 만들 때 트레이트가 사용됩니다. 그렇다면 이때 트레이트는 컴파일러에게 더하기라는 기능이 숫자 타입과 문자열 타입에 적용될 수 있다는 것을 알려주고 더하기라는 기능을 문자열과 숫자 타입에 모두 공유하도록 만들어줍니다. 참고로 트레이트 기능은 타 언어에서 인터페이스(interface)라 불리는 기능과 유사합니다.

Rust는 이 트레이트 기능을 통해 다형성(polymorphism)을 지원합니다. 다형성이란 여러 객체들이 일정한 특성을 공유한다면 이들을 런타임에 서로 바꿔 대입하여 사용할 수 있음을 의미합니다. 다형성에 대한 자세한 설명은 [링크](https://rinthel.github.io/rust-lang-book-ko/ch17-01-what-is-oo.html)를 참조해 주세요.

## 트레이트 선언 방법

Rust에서 트레이트를 선언하는 방법은 `pub trait traitname`를 사용합니다. 아래 예시코드와 같이 선언할 수 있습니다. 아래 예시 코드는 [러스트 프로그래밍 공식 가이드](http://www.yes24.com/Product/Goods/83075894?OzSrank=1) 책을 참조 했습니다.

아래 예시 코드를 보면 함수의 내용이 없다는 특이한 점을 발견할 수 있습니다. 이를 통해 트레이트를 구현할 타입의 행위 즉 메서드의 본문은 반드시 트레이트를 구현하는 각 타입에 의해 구현되어야 함을 알 수 있습니다. 아래의 코드는 Summary 트레이트를 구현하는 모든 타입이 같은 구조의 summarize 메서드를 가지게 끔 보장합니다.

아래 예시 코드의 두 번째 코드와 같이 summarize 메서드의 기능을 비워두는 대신 기본 기능을 추가할 수 도 있습니다. 기본 기능을 사용한다면 타입 별로 별도의 기능 구현 없이도 메서드의 사용이 가능합니다. 단 메서드를 따로 재정의 할 경우 기본 구현 기능은 사용하지 못 하게 된다.

```rust
pub trait Summary{
    fn summarize(&self) -> String;
}

// 기본 기능이 있는 트레이트 
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

## 타입에 따라 트레이트 구현하기

이제 타입에 따라 트레이트가 어떻게 구현되는지 살펴볼 차례입니다. 아래 코드는 NewsArticle과 Tweet이라는 두 구조체 타입에 대해 Summary 트레이트를 구현한 것입니다. `impl 트레이트 이름 for 타입` 구조로 트레이트를 구현하고 그 안에 메서드의 본문을 구현합니다. 타입에 따라 같은 메서드 일지라도 다른 기능을 구현할 수 있습니다. 단 메서드의 구조는 앞서 구현한 트레이트와 동일합니다.

```rust

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

```

## 트레이트 매개변수

트레이트를 함수의 매개변수로도 사용할 수 있습니다. impl 트레이트 이름, 트레이트 경계 정의 문법, 2가지 방법으로 매개변수로 사용할 수 있습니다. 참고로 매개변수가 아니라 트레이트를 리턴하는 함수 또한 작성할 수 있습니다.

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

위 함수에는 Summary 트레이트가 정의된 어떠한 타입이라도 매개변수로 올 수 있습니다. 그러나 정의되지 않은 타입을 전달한다면, 컴파일 에러가 발생합니다. 다음 코드는 트레이트 경계 정의 방법입니다.

```rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}

```

위와 같이 꺽쇠와 :을 통해 지정할 수 있습니다. impl 트레이트 이름 방법의 경우 함수 정의가 간단한 경우 더 편리한 반면 트레이트 경계 문법의 경우 함수 정의가 더 복잡한 경우 사용하는 것이 더 낫습니다.

`+` 문법을 이용해 트레이트의 매개변수를 추가할 수도 있습니다. 이 방법은 위의 두 가지 방법 모두에 적용 가능 합니다. `+` 문법의 예시코드는 다음과 같습니다.

```rust
pub fn notify(item: impl Summary + Display) {
    println!("Breaking news! {}", item.summarize());
}

pub fn notify<T: Summary + Display>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

그러나 `+`로 무한정 트레이트를 추가하는 것도 가독성에 보기 좋지 않습니다. 그럴 때는 where 절을 이용해 정리할 수 있습니다. 아래의 예시코드와 같이 where 절을 사용하여 트레이트 매개변수를 정리할 수 있습니다.

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {}

//정리된 모습
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

위의 두 코드 중 첫 번째 코드에 비해 두 번째 where 절로 정리한 코드가 훨씬 간결해서 가독성이 뛰어남을 알 수 있습니다.
