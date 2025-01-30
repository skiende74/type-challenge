## 7 - Readonly
#### 문제 
객체의 모든 키에 readonly를 붙이는 단순문제
#### 풀이
객체순회시 readonly를 붙여 해결
```ts
type MyReadonly<T> = {readonly[k in keyof T]:T[k]}
```

## 11 - Tuple to Object
#### 문제
튜플을 객체로만드는 유형.
```ts
TupleToObject<typeof (['tesla', 'model 3', 'model X', 'model Y'] as const)>
// expected : { 'tesla': 'tesla', 'model 3': 'model 3', 'model X': 'model X', 'model Y': 'model Y' }
```

#### 풀이
배열순회를 객체로 할 경우이다.
배열(튜플)순회를 객체문법으로 하고싶을 경우 T[number]를 통해순회할 수 있다.

```ts
type TupleToObject<T extends readonly any[]> = {[k in T[number]]: k}
```
([k in keyof T]로 시도하지 말것)

## 8 - Readonly 2
#### 문제
객체에 모두 readonly를 붙이는 게 아니라, 
원하는 키에만 readonly를 붙이고, 나머지 키는 원상태를 유지하여야한다.

#### 풀이
조건부로 readonly를 붙이는 방법은 없다. 따라서 키 리매핑 via as를 활용한다.
readonly를 붙여야할 경우만 골라서 readonly를 붙이고, 나머지 경우는 안붙이고.

```ts
type MyReadonly2<T, K extends keyof T=keyof T> = {readonly[k in keyof T as k extends K? k:never]:T[k]} & {[k in keyof T as k extends K? never:k]:T[k]}
```

독특한 점은 &로 객체를 묶을 경우 기본적으로 동일한 객체로 인식하지 않는다.
하지만 이 문제의 경우 테스트케이스에 ALike<>제네릭으로 조건이 완화되어있으므로 Simplify를 해줄 필요가 없다.

## 9 - Deep Readonly
#### 문제
객체의 Deep Readonly를 구현하여야한다.
다만, 배열의 경우에도 readonly로 바꿔줘야하고(튜플로), 배열내에 있는 객체도 당연히 순회하여야한다.

#### 풀이
배열도 순회해줘야하고, 튜플로도바꿔주는 점, object에 함수도 포함되는데 여기선 무시해야하는점 때문에 조금 성가시다.
조건판단을 잘 해주고, 배열을 mappedType으로 순회하여 간결하게함으로써 해결할 수 있다.
(하지만 잘 생각해보면 배열도 동일한구문으로 순회하므로 따로 조건문을 걸어줄필요가없다.)
즉, readonly를 배열에다 사용하면 튜플이 된다는 것을 알면 된다.

우선 T가 배열인지 판단하고 배열이면, mappedType으로 배열순회를 해주고, readonly를 붙여준다. 
**배열을 mappedType순회시 readonly를 붙이면 튜플로 변한다**
그다음 T가 Function이라면 그냥 T를 리턴,
T가 object라면 {readonly[k in keyof T]: DeepReadonly<T>} 재귀해준다.
그리고 else일때 T리턴.

으로 해결할 수 있다.
하지만 잘 생각해보면 배열도 동일한 구문으로 순회하고싶기때문에, 배열인지여부는 판단할 필요가 없다.
따라서 함수인경우만 검사해주면된다.
```ts
type DeepReadonly<T> = 
T extends Function ? T: { readonly [k in keyof T]: DeepReadonly<T[k]> };
```

## 527 - Append to object
#### 문제
객체에 하나 더 append하는 제네릭구현
```ts
AppendToObject<test3, 'moon', false | undefined> // moon:false|undefined가 추가되어야함
```

#### 풀이
AppendToObject<T, K, V> 라 한다면,
그냥 T에다가 T & { [k in K]:V} 하고 &병합을위해 Simplify해주면된다.
단일값 K로 객체를 만들고싶은 경우도,  {K:V}나 {[K]:V}로는 안되고 **그냥 {[k in K]:V}** 하면 된다.

```ts
Simplify<T> = {[k in keyof T]:T[k]}
AppendToObject<T, K extends string, V> = Simplify<T & {[k in K]: V}>
```

## 1130 - ReplaceKeys
#### 문제
객체에서 K,V(obj)를 넣은내용으로 교체해주는 제네릭 작성하기.
객체는 객체의 유니온일수도있다.
독특한 점은 K에서 골랐으나, V에는 없을경우 never로 해주기.

#### 풀이
[k in keyof T]: 해서 k extends K와 k extends keyof V를 활용해서 해결한다.

k extends K 일 경우,
 k extends keyof V까지 검사해주고 V[k], 
 false면 never
false면 T[k] 사용한다.

```ts
type ReplaceKeys<T, K, V> = 
{ [k in keyof T]: k extends K? k extends keyof V?V[k]:never:T[k]
}
```
