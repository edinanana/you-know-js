> 자바스크립트를 아는것과 제이쿼리를 아는것의 차이
프로그래밍 언어로서 자바스크립트는 끊임없는 비난과 논란의 대상.
수박 겉핥기 정도만 알고 사용해도 웬만큼 서비스 운영이 가능.
적당히 아는건 쉬워도 완전히 알기는 어렵다.
헷갈리는 부분이 나오면 개발자들은 자신의 무지를 탓하기 전에
언어 자체를 비난하곤 하는데 그 나쁜 습관을 바로잡고자 한다


# this 라나 뭐라나
## this 가 뭔가?
### 1. this는 자기자신. 함수 그 자체를 가리킨다?
```js
function foo(num) {
  this.count++;
}
foo.count = 0;

var i;
for (i=0; i<10; i++) {
  if (i > 5) {
    foo(i);
  }
}
```

Q. foo는 몇번 불리고 foo.count 의 값은 뭘까?









_this 에 값이 안쌓이네? 새로운 값에다 넣어야겠다_
```js
var data = {count:0};
function foo(num) {
  data.count++;
}

var i;
for (i=0; i<10; i++) {
  if (i > 5) {
    foo(i);
  }
}
````









해결은 됐지만 this 가 뭔지. 본질을 벗어나 렉시컬 스코프에 의존

그러면 다시 foo 에 있는 count 로 해보자
```js
foo.count
function foo(num) {
  this.count++;
}
foo.count = 0;

var i;
for (i=0; i<10; i++) {
  if (i > 5) {
    foo.call(foo, i);
  }
}
````










이 역시 this를 제대로 이해하지 않은채 문제 회피








그럼 어떻게 쓰면 될까?
스터디 마지막에..






=> this는 자기자신. 함수 그 자체를 가리킨다는 틀린 이야기이다.

### 2. this 는 함수의 스코프를 가리킨다?
```js
function foo() {
  var a = 2;
  this.bar();
}
function bar() {
  return this.a;
}
foo();
````
Q. foo() 의 결과는?








무슨 문제가 있는가?
_넘어야할 산을 넘어버렸음_





this 를 제대로 배우고 싶다면 별의별 오해와 추측은 내버리고
this 는 함수 자신도 아니고 렉시컬 스코프를 가리키는 레퍼런스도 아니라는 점!

this 는 작성 시점이 아닌 호출 시점에 바인딩 됨
함수의 선언 위치와 상관없이 어떻게 함수를 호출했느냐에 따라 정해짐







# this가 이런거로군!
> 내가 this 를 호출하기 전에는
그는 다만
코드 조각에 지나지 않았다
내가 그를 불러다 쓰려고 할 때
그는 나에게로 와서
바인딩 됐다


## 호출부 와 호출 스택
함수를 호출한 지점을 찾아야 this 가 뭘로 바인딩 된건지 알 수 있음
```js
function a() {
  console.log('a');
  b();
}
function b() {
  console.log('b');
  c();
}
function c() {
  console.log('c');
}
a();
````

this 가 뭘로 바인딩 되었는지 찾는 규칙
###1. 기본 바인딩
```js
function foo() {
  "use strict"
  return this.a;
}
var a = 2;
foo();
````
Q. 결과는?






#### strict 모드와의 차이






### 2. 암시적 바인딩
```js
function foo() {
  return this.a;
}

var obj = {
  a: 2,
  foo: foo    // foo 함수를 프로퍼티로 참조하고 있음
};

obj.foo();
````
Q. 결과는?





```js
function foo() {
  return this.a;
}

var obj2 = {
  a: 42,
  foo: foo
};

var obj1 = {
  a: 2,
  obj2: obj2
};

var aa = obj1.obj2.foo;
aa()
````
Q. 결과는?





```js
var a = "global value";

function foo() {
  return this.a;
}

var obj = {
  a: 2,
  foo: foo
};

var bar = obj.foo;

bar();
````
=> 암시적 소실
Implicitly Bound 된 함수에서 바인딩이 소실되는 경우가 있다
bar 가 obj의 foo 를 참조하는 변수처럼 보이지만 X, foo를 직접 가리키는 레퍼런스
bar 가 밖에서 호출되었기 때문에 기본 바인딩 됨
이렇게 애매하게 this 바인딩이 미궁에 빠지는걸 막기 위해 명시적 바인딩이 필요


### 3. 명시적 바인딩
call(), apply()
this 에 바인딩 할 객채를 첫번째 인자로 받아 직접 바인딩

Q. 위 예제를 변경해보자
```js
var a = "global value";

function foo() {
  return this.a;
}

var obj = {
  a: 2,
  foo: foo
};

var bar = obj.foo;

bar.call(obj);
````

// 재사용 가능한 하드 바인딩 패턴
```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = function() {
  return foo.apply( obj, arguments );
};

var b = bar( 3 );
console.log( b );
````
/*
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}
 */

```js
function foo(something) {
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = foo.bind( obj );

bar( 3 );
````


### 4. new 바인딩
자바스크립트는 클래스 지향 언어
클래스 인스턴스 생성시 new 연산자로 호출

#### 함수 앞에 new 를 붙여 호출하면?
- 새 객체가 만들어진다
- 새로 생성된 객체에 prototype 이 연결된다
- 새로 생성된 객체는 함수 호출시 this 로 바인딩 된다
- 자신의 또 다른 객체를 반환하지 않는 한 new 와 함께 호출된 함수는 자동으로 새로 생성된 객체를 반환한다

```js
function foo(a) {
  this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a );
````



정리. this 바인딩 규칙 4가지중
우선순위는? 암시적 바인딩 vs 명시적 바인딩

```js
function foo() {
  return this.a;
}

var obj1 = {
  a: 2,
  foo: foo
};

var obj2 = {
  a: 3,
  foo: foo
};

obj1.foo(); // ?
obj2.foo(); // ?

obj1.foo.call( obj2 ); // ?
obj2.foo.call( obj1 ); // ?
````





암시적 바인딩 vs new 바인딩
```js
function foo(something) {
  this.a = something;
}

var obj1 = {
  foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // ?

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // ?

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // ?
console.log( bar.a ); // ?
````


명시적 바인딩 vs new 바인딩
```js
function foo(something) {
  this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // ?

var baz = new bar( 3 );
console.log( obj1.a ); // ?
console.log( baz.a ); // ?
````


-----------

Dynamic scope? 런타임 도중의 실행 컨텍스트나 호출 컨텍스트에 의해 결정됨
Lexical scope? 소스코드가 작성된 그 문맥에서 결정됨

대부분의 언어는 Lexical scope 규칙을 따름. 자바스크립트 포함.
this 만 빼고


명시적 바인딩
```js
function foo() {
  console.log(this.a);
}

var obj1 = {
  a: 2,
  foo: foo
};

var obj2 = {
  a: 3,
  foo: foo
};

var bar = obj1.foo.call( obj2 );
bar.call(obj2);  // ?
````


ES6 부터 화살표 함수로 표현
위의 4가지 규칙을 따르지 않고 this를 알아서 바인딩 한다
```js
function foo() {
  return (a) => {
    console.log(this.a);
  }
}

var obj1 = { a: 2 };
var obj2 = { a: 3 };
var bar = foo.call(obj1);
bar.call(obj2); // ?
````







화살표 함수는 호출 당시 this를 무조건 어휘적으로 포착
this 를 확실히 보장하는 수단으로 bind() 를 대체할 수 있음  -- 렉시컬 스코프
많이 사용하는 곳이 이벤트 처리기나 콜백 처리

```js
function foo() {
  setTimeout(() => {
    console.log(this.a);
  }, 100);
}

function foo() {
  var self = this;
  setTimeout(function() {
    console.log(self.a);
  }, 100);
}

var obj = { a : 2 };
foo.call(obj);
````
self = this 방법, () => 방법 모두 this 를 제대로 이해하고 수용하기보다 골치아픈 this 에서 도망치려는 꼼수






Q. 그러면 어떻게 해야할까?
```js
function foo() {
  setTimeout(function() {
    console.log(this.a)
  }, 100)
}
var obj = { a : 2 };
foo.call(obj);
````



## 정리하기
 this 바인딩은 함수의 직접적인 호출부에 따라 달라진다
 일단 호출부를 식별한 후 4가지 규칙에 따라 우선순위 적용한다

 1. new 로 호출했다면 새로 생성된 객체로 바인딩 (new 바인딩)
 2. call 이나 apply 또는 bind 로 호출됐다면 주어진 객체로 바인딩 (명시적 바인딩)
 3. 객체를 포함하는 형태로 호출됐다면 해당 객체로 바인딩 (암시적 바인딩)
 4. 그외는 기본값 - 엄격모드는 undefined, 비엄격모드는 window (기본 바인딩)


 ES6의 화살표 함수는 표준 바인딩 규칙을 무시하고 렉시컬 스코프로 this를 바인딩 하므로 삼가하자

