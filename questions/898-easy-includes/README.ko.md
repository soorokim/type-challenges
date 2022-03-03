<!--info-header-start--><h1>Includes <img src="https://img.shields.io/badge/-%EC%89%AC%EC%9B%80-7aad0c" alt="쉬움"/> <img src="https://img.shields.io/badge/-%23array-999" alt="#array"/></h1><blockquote><p>by null <a href="https://github.com/kynefuk" target="_blank">@kynefuk</a></p></blockquote><p><a href="https://tsch.js.org/898/play/ko" target="_blank"><img src="https://img.shields.io/badge/-%EB%8F%84%EC%A0%84%ED%95%98%EA%B8%B0-3178c6?logo=typescript&logoColor=white" alt="도전하기"/></a> &nbsp;&nbsp;&nbsp;<a href="./README.md" target="_blank"><img src="https://img.shields.io/badge/-English-gray" alt="English"/></a>  <a href="./README.zh-CN.md" target="_blank"><img src="https://img.shields.io/badge/-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-gray" alt="简体中文"/></a> </p><!--info-header-end-->

JavaScript의 `Array.includes` 함수를 타입 시스템에서 구현하세요. 타입은 두 인수를 받고, `true` 또는 `false`를 반환해야 합니다.

예시:

```ts
type isPillarMen = Includes<['Kars', 'Esidisi', 'Wamuu', 'Santana'], 'Dio'> // expected to be `false`
```

풀이:
  - 문제점: Record 형태 일때 property의 readonly속성 까지 체크 하지는 못했다. 
  - easy를 못푸는것이 너무 분해서 8시간 동안 잡고 있다가 답안을 봤..는데 어려운 개념이 필요한 부분이었던것 같다. 
          

```ts
type CheckRecord<T, U extends Record<RecordKey,unknown>> = T extends infer A ? U extends infer B ? B extends A ? true : false : never : never
type IsRecord<T> = T extends Record<RecordKey, any> ? true : false 
type IsEqual<T,U> = T extends infer A ? U extends infer B ? A extends B ? IsRecord<B> extends true ? CheckRecord<A,B> : true:false : never : never
type Includes<T extends readonly any[], U> = 
    T extends [infer V, ...infer X] ? IsEqual<V, U> extends true ? true 
    : (X['length'] extends 0 ? false : Includes<X,U>)
    : neve
```


답안:
https://github.com/type-challenges/type-challenges/issues/1568

```ts
export type IsEqual<X, Y> =
    (<T>() => T extends X ? 1 : 2) extends
    (<T>() => T extends Y ? 1 : 2) ? true : false;


type Includes<Value extends any[], Item> =
	IsEqual<Value[0], Item> extends true
		? true
		: Value extends [Value[0], ...infer rest]
			? Includes<rest, Item>
			: false;
```

테스트:
  어떻게 IsEqual이 동작 할 수 있는지 이해가 되지 않아 테스트를 진행함
  결과적으로는 서로다른 제네릭으로 타입을 만들어 비교를 해야 readonly까지 체크 되는것을 발견 
  test33 과 test44를 같은 제네릭으로 만들고 비교를 했을시 true가 나와 체크를 할 수 없는것으로 확인
  

```ts
  type TestData = {
    readonly a: '2'
  }
  type TestData2 = {
    a: '2'
  }

  type TestGeneric<T> = <X>()=> X extends T ? 1:2 
  type TestGeneric2<T> = <X>()=> X extends T ? 1:2 

  let test11:TestGeneric<TestData> = {} as TestGeneric<TestData>
  let test22:TestGeneric2<TestData2> = {} as TestGeneric2<TestData2>
  type T12 = typeof test11 extends typeof test22 ? true:false // <- false

  let test33:TestGeneric<TestData> = {} as TestGeneric<TestData>
  let test44:TestGeneric<TestData2> = {} as TestGeneric<TestData2>
  type T34 = typeof test33 extends typeof test44 ? true:false // <- true

```

case: 같은 제네릭을 사용하더라도 타입이 아예 다르면 체크가 됨
```ts
  let test33:TestGeneric<number> = {} as TestGeneric<number>
  let test44:TestGeneric<string> = {} as TestGeneric<string>
  type T34 = typeof test33 extends typeof test44 ? true:false // <- false
  
  let test33:TestGeneric<{a:'a'}> = {} as TestGeneric<{a:'a'}>
  let test44:TestGeneric<{a:'b'}> = {} as TestGeneric<{a:'b'}>
  type T34 = typeof test33 extends typeof test44 ? true:false // <- false
```
case: Readonly를 넣어도 체크가 되지 않음 
```ts
  let test33:TestGeneric<Readonly<{a:string}>> = {} as TestGeneric<Readonly<{a:string}>>
  let test44:TestGeneric<{a:string}> = {} as TestGeneric<{a:string}>
  type T34 = typeof test33 extends typeof test44 ? true:false // <- false
```
이런 일 도 일어날 수 있다.
<br>
<img width="462" alt="image" src="https://user-images.githubusercontent.com/44357561/156214195-bb1c358b-b843-4c8e-aa07-d96dfae6b6fa.png">

<!--info-footer-start--><br><a href="../../README.ko.md" target="_blank"><img src="https://img.shields.io/badge/-%EB%8F%8C%EC%95%84%EA%B0%80%EA%B8%B0-grey" alt="돌아가기"/></a> <a href="https://tsch.js.org/898/answer/ko" target="_blank"><img src="https://img.shields.io/badge/-%EC%A0%95%EB%8B%B5%20%EA%B3%B5%EC%9C%A0%ED%95%98%EA%B8%B0-teal" alt="정답 공유하기"/></a> <a href="https://tsch.js.org/898/solutions" target="_blank"><img src="https://img.shields.io/badge/-%EC%A0%95%EB%8B%B5%20%EB%B3%B4%EA%B8%B0-de5a77?logo=awesome-lists&logoColor=white" alt="정답 보기"/></a> <!--info-footer-end-->
