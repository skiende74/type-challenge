## 3312 - Parameters
#### 문제
함수의 parameters를 얻는 제네릭 생성하기
#### 풀이
extends문에서 infer하여 사용한다.
제네릭 인자로 받을 땐 (never[])=>unknown으로 하면 가장 넓으므로 이렇게해준다. (그냥 any로 처리해도 상관없긴함)

```ts
type MyParameters<T extends (...args: any[]) => any> = T extends (...args: infer A)=>any?A:never;
```

## 2 - ReturnType
#### 문제
```
Equal<123, MyReturnType<(a:number) => 123>>
```
함수의 리턴타입얻기
#### 풀이
리턴타입을 얻어내는 문제이다. infer이용하여 리턴타입을 얻으면 된다.
매개변수의 타입을 어떻게 줄지가 중요한데,
(...args:unknown[])으로 줬다가는 매칭이 안된다고 never로 가버린다.
함수타입의 측면에서 반공변인 매개변수의 unknown[]은 너무 좁기때문이다(넓을수록 좁은타입).
따라서 가장좁은타입인 never[]로 줘야한다.

```ts
type MyReturnType<T> = T extends (...args:never[])=>infer R?R:never;
```

## 10 - Tuple To Union
#### 문제
튜플을 유니온으로 바꾸기

#### 풀이
그냥 튜플의 [number]를 쓰면되는 단순문제
```ts
type TupleToUnion<T extends unknown[]> = T[number]
```

## 2070 - Drop Char
#### 문제
```ts
Equal<DropChar<'    butter fly!        ', ' '>, 'butterfly!'>
```
#### 풀이
중간에 C를 만나면 제거해야한다.
`${infer A}${C}${infer B}`로 매칭을한다. 이렇게하면 A는 빈문자열도 가능하다. 
infer A : 빈문자열가능
infer A extends string : 빈문자열 불가

따라서 저 로직은 C를제거해준다.
또 그 결과 `${A}${DropChar<B,C>}`만 해주면된다. 왜냐하면 왼쪽부터 매칭을해나가기때문에 A에는 C가없는것이 보장된다.
```ts
type DropChar<S,C extends string = ' '> = S extends `${infer A}${C}${infer B}`?`${A}${DropChar<B, C>}`:S
```

## 9616 - ParseUrlParams
#### 문제
```ts
Equal<ParseUrlParams<':id'>, 'id'>
Equal<ParseUrlParams<'posts/:id/:user/like'>, 'id' | 'user'>
```
첫예시처럼 앞에 슬래쉬가 없는경우도있고, 슬래쉬로구분이될때도있고해서 깔끔하게하기가 조금 귀찮았다.

#### 풀이
하나의 extends로 깔끔하게하려하니 안풀렸다. 근데 그냥 extends 2개쓰면 된다.
너무 깔끔하게 할려고 집착안하고 좀더길게써도되지않을까
( 근데 푸는 입장에선 그 기준을 잡을수가없다. 어느정도 감을 모두잡기전까진 어쩔수없는듯 )

```ts
type ParseUrlParams<T> = 
T extends `${string}:${infer Rest}`
? Rest extends `${infer L}/${infer R}`
  ? L | ParseUrlParams<R>
  : Rest
: never
```

## 62 - Type LookUp
#### 문제

#### 풀이
U가 제네릭이므로 제네릭유니온의 분배법칙을활용하여 순회없이풀수있다.
```ts
type LookUp<U extends {type: string}, T extends Animal['type']> = U extends {type:T}? U:never;
```
여기서 U,T에 extends로 조건을 걸어주지않아도 문제는 풀리지만,
유저의 실수를 막아주기위해 U, T를 좁혀주는 게 좋을 것이다.

또, 유니온자체는 분배되지만, 유니온에 ['type']을 걸면 분배되지않는다.
즉, U['type'] extends T?로는 오답이다.

## 5821 - MapTypes
#### 문제
```ts
type StringToNumber = { mapFrom: string; mapTo: number;}
type StringToDate = { mapFrom: string; mapTo: Date;}
MapTypes<{iWillBeNumberOrDate: string}, StringToDate | StringToNumber> // gives { iWillBeNumberOrDate: number | Date; }
```
위와같이 객체에대해 매핑을 지원하는것. 제네릭도 지원해야한다는점.

#### 풀이
`type MapTypes<T, R>`
정리하자면 T에대해서도 순회를해야하고, R에대해서도 순회를해야한다.

우선 T 객체순회를 하고([k in keyof T]), 
객체 내에서 R extends 분배법칙을하여 R순회를 해준다. (R extends는 당연히 객체 안에서 써줘야 된다. {a:c|d}와 {a:c}|{a:d}의 차이. {a:c|d}를 원하므로)
그럼 `R extends {mapFrom: T[k]}?` 가 될것이고,
만약 참이라면 `R['mapTo']`, 거짓이면 `never`를 리턴.

하지만 이렇게만하면, 매칭되는게없을때 T[k]가 되어야하는데도 never가 되어버리는문제가있다.
그렇다고 never를 T[k]로 바꿔버리면 T[k]가 무조건포함되어버리고.

이 문제를 해결하기위해
1. 로직을 한번더 반복하여, 결과가 never면 T[k]로 바꿔치기
2. 그냥 never가 될지만을 검사하기

가 있겠는데, 당연히 2번이 더 좋은 방법일것이다.
never가 되는지여부는 `R['mapFrom'] extends T[k]`로 검사가 가능하다.
이를 식으로 옮기면 아래와 같다.

```ts
type MapTypes<T, R extends {mapFrom:any; mapTo:any}> = 
{ [k in keyof T]: 
    T[k] extends R['mapFrom']
    ? R extends {mapFrom: T[k]}
      ? R['mapTo']
      : never
    : T[k]; 
}
```
