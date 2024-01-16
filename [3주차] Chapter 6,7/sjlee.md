# 6. 고급타입

## 알아야할 내용들
- 서브타입화
- 할당성, 가변성, 넓히기
- 정제, 종합성
- 객체 타입을 키로 활용하고 매핑하는 방법
- 조건부 타입 사용
- 자신만의 타입 안전 장치 정의
- 타입 assertion
- 확실한 할당 assertion
- 타입 안전성을 높일 수 있는 방법들
  - 컴패니언 객체 패턴
  - 튜플 타입의 추론 개선
  - 이름 기반 타입 흉내내기
  - 안전하게 프로토타입 확장하기

## 슈퍼 타입 vs 서브타입

- A와 A1가 있고 A가 A1의 서브타입이면 A1이 필요한 곳이면 어디든 A를 안전하게 사용할 수 있다

| 슈퍼타입 | 서브타입 |
| :------: | :------: |
|   객체   |   배열   |
|   배열   |   튜플   |
|   any    | 모든 것  |
| 모든 것  |  never   |

#### 슈퍼 타입(부모) <-> 서브 타입(자식)

### 가변성이란

- 타입 호환이 되려면 지정된 타입의 서브타입(자식)이 필요
- 즉 지정된 타입을 모두 포함하고 있는 타입이 와야함

### 타입 넓히기

- as const 를 사용하면 타입 넓히기가 중지되고 멤버들 까지 자동으로 readonly가 된다
  - 중첩된 자료구조에도 재귀적으로 적용된다

### 정제

- 함수안의 if문에서 특정 타입일 때 return 시킴으로써 그 아래 코드들은 해당 타입을 제외하는 것으로 추론
  - null 또는 undefined일 때 early return할 때 주로 사용

## 객체 타입의 타입 연산자

#### key in

```ts
type APIResponse = {
  user: {
    userId: string;
    friendList: {
      count: number;
      friends: {
        firstName: string;
        lastName: string;
      }[];
    };
  };
};

type FriendList = APIResponse["user"]["friendList"];
```

#### keyof

- 객체의 모든 키를 문자열 리터럴 타입 유니온으로!

```ts
type UserKeys = keyof APIResponse["user"]; // "userId" | "friendList";
```

- tsconfig에서 keyofStringsOnly를 활성화하면 string으로 간주
  - 아니면 string | number | Symbol

### Record 타입

```ts
type Weekday = "Mon" | "Tue" | "Wed" | "Thu" | "Fri";
type Day = Weekday | "Sat" | "Sun";

const nextDay: Record<Weekday, Day> = {
  Mon: "Tue",
};

// Mapped type
const nextDay1: { [key in Weekday]: Day } = {
  Mon: "Tue",
};
```
- Mapped type은 Record보다 강력함
  - 키인 타입과 조합하면 키 이름별로 매핑할 수 있는 값 타입을 제한할 수 있기 때문

```ts
/* ** Mapped type 예시 ** */
type Account = {
  id: number;
  isEmployee: boolean;
  notes: string[];
};

// 모든 필드를 선택형
type OptionalAccount = {
  [K in keyof Account]?: Account[K];
};

// 모든 필드를 nullable
type NullableAccount = {
  [K in keyof Account]: Account[K] | null;
};

// 모든 필드를 읽기 전용
type ReadonlyAccount = {
  readonly [K in keyof Account]: Account[K];
};

// 모든 필드를 다시 쓸수 있도록
type Account2 = {
  -readonly [K in keyof ReadonlyAccount]: Account[K]; // -를 붙이는 것은 처음보네
};

// 모든 필드를 다시 필수형으로
type Account3 = {
  [K in keyof OptionalAccount]-?: Account[K];
};
```

### 내장 매핑된 타입

`Record<keys, Values>`

- key 타입과 values 타입의 값을 갖는 객체

`Partial<Obj>`

- Obj의 모든 필드를 선택형으로 표시

`Required<Obj>`

- Obj의 모든 필드를 필수형으로 표시

`Readonly<Obj>`

- Obj의 모든 필드를 읽기 전용으로 표시

`Pick<Obj, Keys>`

- 주어진 Keys에 대응하는 Object의 서브타입을 반환

```ts
// Pick
type Student = {
  id: number;
  name: string;
  age: number;
};
const student: Pick<Student, "id" | "name"> = {
  id: 1,
  name: "Tony",
};
// type Pick<T, K extends keyof T> = { [P in K]: T[P]; }
```

| Obj 타입 | Union 타입 |
| :------: | :--------: |
|   Pick   |  Extract   |
|   Omit   |  Exclude   |

### 컴패니언 객체 패턴

- 같은 이름을 공유하는 객체와 클래스를 쌍으로 연결
  - 스칼라에서 유래
  - e.g., Class
    - 타입과 값은 별도의 네임스페이르를 갖는데 클래스는 컴패니언 객체 패턴이 적용되어
    - 값과 타입이 같은 이름으로 한번에 정의 됨

```ts
type Currency = {
  unit: "EUR" | "GBP" | "JPY" | "USD";
  value: number;
};

let Currency = {
  DEFAULT: "USD",
  from(value: number, unit = Currency.DEFAULT): Currency {
    return { unit, value };
  },
};

// class Currency
// ...
```

### 튜플의 타입 추론 개선

```ts
// bad
const aTuple = [a, true]; // (number | boolean)[]

// good
function tuple<T extends unknown[]>(...t: T) {
  return t;
}

const bTuple = tuple(1, true); // [number, boolean]
```

### 사용자 정의 타입 안전 장치

```ts
// is : 리턴 시 파라미터의 타입을 지정 - type-predicates
// https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates
function isString(a: unknown): a is string {
  return typeof a === "string";
} // return 타입은 boolean이지만 그 때 a는 string으로 좁혀진다

function parseInput(input: string | number) {
  let formattedInput: string;
  if (isString(input)) {
    formattedInput = input.toUpperCase(); // string에만 존재하는 toUpperCase()를 사용해도 에러가 발생하지 않음
  }
}
```

### 조건부 타입

```ts
type IsString<T> = T extends string ? true : false;
```

### 분배적 조건부

```ts
type ToArray<T> = T extends unknown ? T[] : T[];
type A = ToArray<number>;
type B = ToArray<number | string>;
```

```ts
type Without<T, U> = T extends U ? never : T;
type WithoutA = Without<boolean | number | string, boolean>; // number | string
```

### infer 키워드

- 조건의 일부를 제네릭 타입으로 선언
  - 제네릭을 인라인으로 선언하는 문법 : infer

```ts
// 6.5.2 infer
type ElementType<T> = T extends unknown[] ? T[number] : T;
type A = ElementType<number[]>; // number
type B = ElementType<string[]>; // string
type C = number extends unknown[] ? true : false;
type D = number[][number]; // number

// infer로 표현하기
type ElementType2<T> = T extends (infer U)[] ? U : T; // 이해가 잘 안됨 안쓸 듯
```

### 내장 조건부 타입들

```ts
type A = number | string | boolean;
type B = string | boolean;
type C = Exclude<A, B>; // number
```

- Exclude => 제거

```ts
type A = number | string | boolean;
type B = string;
type C = Extract<A, B>; // string
```

- Extract => 추출

### 타입 어서션

- `as`, 단언, 슈퍼타입이나 서브타입으로만 어서션할 수 있다

### Nonnull 어서션

- null 또는 undefined이 아님을 단언

```ts
element.parentNode!.removeChild(element); // 옵셔널 체이닝 처럼 느낌표를 넣어서 nonnull임을 표시
```

- nonnull 어서션을 많이 사용하고 있다면 코드를 리팩터링해야한다는 징조일 수 있다

### 확실한 할당 어서션

```ts
let userId!: string; // ?와 같이 !를 타입 선언 시 식별자 뒤에 붙이면 확실하게 할당되어 있을 것을 의미(nonnull)
```

# 7. 에러 처리
- 에러를 표현하고 처리하는 가장 일반적인 패턴 4가지
  - null 반환
  - 예외 던지기
  - 예외 반환
  - Option 타입

## null 반환

- 장점 : 에러 상황에 대한 대처
- 단점
  - 문제가 생긴 원인을 알 수 없음
    - 개발자가 작성한 모호한 에러메세지를 보게 되기 때문
  - null반환 시 null확인 처리를 해야되기 때문에 코드가 지저분해질 수 있다

## 예외 던지기

- 가능한 null반환 대신 예외를 던지자

  - 어떤 문제인지에 대한 메타데이터(에러 메세지)를 얻을 수 있다

- try - catch로 에러를 잡아서 처리
  - 예상한 예외가 아닌 에러는 다시 던지는 것이 좋다

```ts
try {
  const date = parse();
  console.info("Date is", date.toISOString());
} catch (e) {
  if (e instanceof RangeError) {
    console.error(e.message);
  } else {
    throw e;
  }
}
```

RangeError를 상속받아서 커스텀 에러를 만들어 에러를 구분하게 할 수도 있다

```ts
class InvalidDateFormatError extends RangeError {}
class DateIsTheFutureError extends RangeError {}
```

타입스크립트의 예외는 never 타입(취급하지 않음)

- docblock에 정보를 추가해야 함

```ts
/**
 * @throw {InvalidDateFormatError} 사용자가 생일을 잘 못 입력함
 * @throw {DateIsTheFutureError} 사용자가 생일을 미래 날짜로 입력함
 */
const parse = (birthday: string) => {
  const date = new Date(birthday);
  if (!isValid(date)) {
    throw new InvalidDateFormatError("Invalid date format");
  }
  if (date.getTime() > Date.now()) {
    throw new DateIsTheFutureError("Date is in the future");
  }
  return date;
};
```

## 7.3 예외 반환

- 타입스크립트는 자바가 아니며 throws문을 지원하지 않는다.
- 하지만 유니온 타입을 이용해 비슷하게 흉내낼 수 있다
  - 리턴타입에 에러 타입을 명시해준다

```ts
const parse = (
  birthday: string
): Date | InvalidDateFormatError | DateIsTheFutureError => {
  const date = new Date(birthday);
  if (!isValid(date)) {
    return new InvalidDateFormatError("Invalid date format");
  }
  if (date.getTime() > Date.now()) {
    return new DateIsTheFutureError("Date is in the future");
  }
  return date;
};

const result = parse("123ㅁㅁ");
if (result instanceof InvalidDateFormatError) {
  console.error(result); // 에러 객체 전체 출력
  // console.error(result.message);
} else if (result instanceof DateIsTheFutureError) {
  console.info(result.message);
} else {
  console.info("Date is", result.toISOString());
}

// 위가 귀찮다면 -> 모든 에러 커버
const result = parse();
if (result instanceof Error) {
  console.error(result.message);
} else {
  console.info("Date is", result.toISOString());
}
```

- 위에서 수행한 것들

  - parse의 시그니처에 발생할 수 있는 예외를 나열했다
  - 메서드 사용자에게 어떤 에러가 발생할 수 있는지를 전달했다
  - 메서드 사용자가 각각의 에러를 모두 처리하거나 다시 던지도록 강제했다

- 이 방식은 단순하고 API 실패 유형과 에러를 알려주기에 충분하다

```ts
function canMakeError(): T | Error1 {
  // ...
}

function func1WithCanMakeError(): U | Error1 | Error2 {
  const resultOfCanMakeError = canMakeError();
  if (resultOfCanMakeError instanceof Error) {
    return resultOfCanMakeError;
  }
  // resultOfCanMakeError로 어떤 동작을 수행(정상동작)
}
```

- 이 방식은 조금 복잡한 대신 안전성이 뛰어나다

### Option 타입

- 특수 목적 데이터 타입을 사용해 예외를 표현하는 방법도 있다
#

- 타입스크립트에서 에러 처리에 대한 여러가지 방법
  - null 반환
  - 예외 던지기
  - 예외 반환하기
  - Option 타입
- 어떤 방법을 사용할지는 다음을 기준으로 정하면 된다
  - 어떤 작업이 실패했음을 단순하게 알리기 => null, Option
  - 실패한 이유와 관련 정보를 제공 => 예외를 던지거나 반환
  - 가능한 모든 예외를 사용자가 명시적으로 처리하도록 강제 => 예외 반환
  - 에러 처리 관련 코드를 더 적게 구현 => 예외 던지기
  - 에러를 만드는 방법이 필요하다 => Option
  - 단순히 에러가 발생했을 때 처리 => null, 예외

1. TS에서 서브타입에 대한 삭제 즉, 파괴적 갱신은 불가능하다 (O/X)
2. 함수의 매개변수와 this 타입은 기본적으로 반변이다. (O/X)
3. null 이나 undefined로 초기화된 변수의 타입은?
4. 객체를 신선한 객체로 보고, 초과 프로퍼티 확인을 수행하는 것은?
5. tag된 유니온이 중요한 이유는? 어디에 쓰이나요?
6. keyof 연산자는 무엇인가요?
7. 모든 종류의 타입을 담을 수 있는 타입은?
8. 에러처리 일반적인 방법 3가지와 각 에러 처리의 단점을 말해주세요!
9. Option은 어떤 타입인가요?
10. type assertion이란?
