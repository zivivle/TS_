## Zod docs 정리

### 기본 사용법

#### string 스키마 만들기

```ts
import { z } from "zod";

// 문자열 스키마 생성
const mySchema = z.string();

// parse
mySchema.parse("tuna"); // => "tuna"
mySchema.parse(12); // => ZodError 발생

// safeParse (유효성 검증 실패 시 오류를 던지지 않음)
mySchema.safeParse("tuna"); // => { success: true; data: "tuna" }
mySchema.safeParse(12); // => { success: false; error: ZodError }
```

#### object 스키마 만들기

```ts
import { z } from "zod";

const User = z.object({
  username: z.string(),
});

User.parse({ username: "Ludwig" });

// 추론된 타입 추출
type User = z.infer<typeof User>;
// { username: string }
```

#### 원시 타입

```ts
import { z } from "zod";

// 원시 값들
z.string();
z.number();
z.bigint();
z.boolean();
z.date();
z.symbol();

// 빈 타입들
z.undefined();
z.null();
z.void(); // undefined 허용

// 모든 값을 허용하는 catch-all 타입들
z.any();
z.unknown();

// 값이 없는 never 타입
z.never();
```

#### 원시 값에 대한 변환(Coercion)

Zod는 원시 값을 변환하는 더 편리한 방법을 제공한다.

```ts
const schema = z.coerce.string();
schema.parse("tuna"); // => "tuna"
schema.parse(12); // => "12"
```

parse시 입력 값은 String() 함수(자바스크립트 내장 함수)를 통해 문자열로 변환된다.

```ts
schema.parse(12); // => "12"
schema.parse(true); // => "true"
schema.parse(undefined); // => "undefined"
schema.parse(null); // => "null"
```

반환된 스키마는 일반적인 ZodString 인스턴스이므로 모든 문제열 메서드를 사용할 수 있다.

```ts
z.coerce.string().email().min(5);
```

#### 변환 작동 방식

모든 원시 타입은 변환을 지원한다. Zod는 String(input), Number(input), new Date(input) 등 자바스크립트 내장 생성자를 사용해 모든 입력값을 변환한다.

```ts
z.coerce.string(); // String(input)
z.coerce.number(); // Number(input)
z.coerce.boolean(); // Boolean(input)
z.coerce.bigint(); // BigInt(input)
z.coerce.date(); // new Date(input)
```

#### 주의사항

x.coercce.boolean()을 사용할 때, 예상과 다르게 작동할 수 있다. 모든 truthy 값은 true로, 모든 falsy값은 false로 변환된다.

```ts
const schema = z.coerce.boolean(); // Boolean(input)

schema.parse("tuna"); // => true
schema.parse("true"); // => true
schema.parse("false"); // => true
schema.parse(1); // => true
schema.parse([]); // => true

schema.parse(0); // => false
schema.parse(""); // => false
schema.parse(undefined); // => false
schema.parse(null); // => false
```

변환 로직에 더 많은 제어가 필요하다면, z.preprocess 또는 z.pipe()를 사용할 수 있다.

#### 리터럴 (Literals)

리터럴 스키마는 "hello world" 또는 5와 같은 리터럴 타입을 나타낸다.

```ts
const tuna = z.literal("tuna");
const twelve = z.literal(12);
const twobig = z.literal(2n); // bigint 리터럴
const tru = z.literal(true);

const terrificSymbol = Symbol("terrific");
const terrific = z.literal(terrificSymbol);

// 리터럴 값 가져오기
tuna.value; // "tuna"
```

현재 Zod는 Date 리터럴은 지원하지 않는다.

#### 문자열 (String)

Zod는 문자열에 대한 다양한 유효성 검사를 포함한다.

```ts
// 유효성 검사
z.string().max(5); // 최대 길이 5
z.string().min(5); // 최소 길이 5
z.string().length(5); // 정확히 5자
z.string().email(); // 이메일 형식
z.string().url(); // URL 형식
z.string().emoji(); // 이모지 포함
z.string().uuid(); // UUID 형식
z.string().nanoid(); // 나노ID
z.string().cuid(); // CUID 형식
z.string().cuid2(); // CUID2 형식
z.string().ulid(); // ULID 형식
z.string().regex(regex); // 정규표현식 검사
z.string().includes(string); // 특정 문자열 포함
z.string().startsWith(string); // 특정 문자열로 시작
z.string().endsWith(string); // 특정 문자열로 끝남
z.string().datetime(); // ISO 8601 형식 (기본값: `Z` 타임존만 허용)
z.string().ip(); // IPv4 및 IPv6 허용 (기본값)
```

#### 변환 (Transforms)

```ts
z.string().trim(); // 공백 제거
z.string().toLowerCase(); // 소문자로 변환
z.string().toUpperCase(); // 대문자로 변환
```

### 커스텀 에러 메세지

유효성 검사 메서드를 사용할 때 인자로 커스텀 에러 메세지를 제공할 수 있다.

```ts
z.string().min(5, { message: "최소 5자 이상이어야 합니다." });
z.string().max(5, { message: "최대 5자 이하이어야 합니다." });
z.string().email({ message: "유효하지 않은 이메일 주소입니다." });
```

#### 날짜와 시간 (Datetimes)

Zod는 몇 가지 날짜 및 시간 관련 유효성 검사를 제공한다. 기본적으로 ISO 8610 형식을 따른다.

```ts
const datetime = z.string().datetime({ offset: true });
datetime.parse("2020-01-01T00:00:00+02:00"); // 허용
```

#### IP 주소(IP Address)

Zod는 기본적으로 IPv4 및 IPv6를 지원한다.

```ts
const ip = z.string().ip();
ip.parse("192.168.1.1"); // 허용
```

#### 숫자 (Numbers)

숫자 스키마를 만들 때 특정 에러 메세지를 커스터마이즈할 수 있다.

```ts
const age = z.number({
  required_error: "나이는 필수입니다.",
  invalid_type_error: "나이는 숫자여야 합니다.",
});
```

Zod는 숫자와 관련된 다양한 유효성 검사를 제공한다.

```ts
z.number().gt(5); // 5보다 커야 함
z.number().gte(5); // 5 이상 (alias: .min(5))
z.number().lt(5); // 5보다 작아야 함
z.number().lte(5); // 5 이하 (alias: .max(5))

z.number().int(); // 정수여야 함

z.number().positive(); // 양수여야 함 (> 0)
z.number().nonnegative(); // 0 이상이어야 함 (>= 0)
z.number().negative(); // 음수여야 함 (< 0)
z.number().nonpositive(); // 0 이하여야 함 (<= 0)

z.number().multipleOf(5); // 5로 나누어 떨어져야 함 (alias: .step(5))

z.number().finite(); // 유한 값이어야 함 (Infinity나 -Infinity는 허용되지 않음)
z.number().safe(); // 안전한 숫자여야 함 (Number.MIN_SAFE_INTEGER와 Number.MAX_SAFE_INTEGER 사이)
```

선택적으로 두 번째 인자를 전달하여 커스텀 에러 메시지를 설정할 수 있다.

```ts
z.number().lte(5, { message: "값이 너무 큽니다." });
```

#### BigInts

Zod는 BigInt에 대한 다양한 유효성 검사도 제공한다.

```ts
z.bigint().gt(5n); // 5n보다 커야 함
z.bigint().gte(5n); // 5n 이상 (alias: .min(5n))
z.bigint().lt(5n); // 5n보다 작아야 함
z.bigint().lte(5n); // 5n 이하 (alias: .max(5n))

z.bigint().positive(); // 양수여야 함 (> 0n)
z.bigint().nonnegative(); // 0n 이상이어야 함 (>= 0n)
z.bigint().negative(); // 음수여야 함 (< 0n)
z.bigint().nonpositive(); // 0n 이하여야 함 (<= 0n)

z.bigint().multipleOf(5n); // 5n으로 나누어 떨어져야 함
```

#### NaN (Not a Number)

NaN 스키마를 만들 때도 특정 에러 메세지를 커스터마이즈할 수 있다.

```ts
const isNaN = z.nan({
  required_error: "isNaN는 필수입니다.",
  invalid_type_error: "isNaN는 '숫자가 아님'이어야 합니다.",
});
```

#### Booleans

Boolean 스키마도 커스터마이즈가 가능하다.

```ts
const isActive = z.boolean({
  required_error: "활성 상태는 필수입니다.",
  invalid_type_error: "활성 상태는 Boolean 값이어야 합니다.",
});
```

#### Dates

z.date()를 사용하여 날자 인스턴스를 검증할 수 있다.

```ts
z.date().safeParse(new Date()); // 성공: true
z.date().safeParse("2022-01-12T00:00:00.000Z"); // 실패: false
```

날짜 스키마를 만들 때 커스텀 에러 메세지를 설정할 수 있다.

```ts
const myDateSchema = z.date({
  required_error: "날짜와 시간을 선택해 주세요.",
  invalid_type_error: "유효한 날짜가 아닙니다!",
});
```

Zod는 날짜와 관련된 다양한 유효성 검사도 제공한다.

```ts
z.date().min(new Date("1900-01-01"), { message: "너무 오래된 날짜입니다." });
z.date().max(new Date(), { message: "너무 이전 날짜입니다." });
```

#### Enums

z.enum()은 고정된 문자열 값 집합을 허용하는 스키마를 선언하는 Zod 고유의 방법이다.

```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);
type FishEnum = z.infer<typeof FishEnum>; // 'Salmon' | 'Tuna' | 'Trout'
```

FishEnum.enum.Salmon으로 자동 완성이 가능하다.
또한 .options 속성을 통해 옵션 목록을 튜플로 가져올 수 있다.

```ts
FishEnum.options; // ["Salmon", "Tuna", "Trout"]
```

z.enum()은 고정된 값의 서브셋을 생성하는 .exclude와 .exract 메서드를 제공한다.

```ts
const SalmonAndTrout = FishEnum.extract(["Salmon", "Trout"]);
const TunaOnly = FishEnum.exclude(["Salmon", "Trout"]);
```

#### Native Enums

Zod Enums는 Enums을 정의하고 검증하는 데 권장되는 방식이다. 하지만 다른 라이브러리의 Enums를 검증해야 하거나, 기존 열거형을 다시 작성하고 싶지 않을 경우, z.nativeEnum()을 사용할 수 있다.

**Numeric Enums**

```ts
enum Fruits {
  Apple,
  Banana,
}

const FruitEnum = z.nativeEnum(Fruits);
type FruitEnum = z.infer<typeof FruitEnum>; // Fruits

FruitEnum.parse(Fruits.Apple); // 통과
FruitEnum.parse(Fruits.Banana); // 통과
FruitEnum.parse(0); // 통과
FruitEnum.parse(1); // 통과
FruitEnum.parse(3); // 실패
```

**String Enums**

```ts
enum Fruits {
  Apple = "apple",
  Banana = "banana",
  Cantaloupe, // 숫자형과 문자열형 열거형 혼합 가능
}

const FruitEnum = z.nativeEnum(Fruits);
type FruitEnum = z.infer<typeof FruitEnum>; // Fruits

FruitEnum.parse(Fruits.Apple); // 통과
FruitEnum.parse(Fruits.Cantaloupe); // 통과
FruitEnum.parse("apple"); // 통과
FruitEnum.parse("banana"); // 통과
FruitEnum.parse(0); // 통과
FruitEnum.parse("Cantaloupe"); // 실패
```

**Const Enums**
z.nativeEnum() 함수는 as const 객체에서도 작동한다.

```ts
const Fruits = {
  Apple: "apple",
  Banana: "banana",
  Cantaloupe: 3,
} as const;

const FruitEnum = z.nativeEnum(Fruits);
type FruitEnum = z.infer<typeof FruitEnum>; // "apple" | "banana" | 3

FruitEnum.parse("apple"); // 통과
FruitEnum.parse("banana"); // 통과
FruitEnum.parse(3); // 통과
FruitEnum.parse("Cantaloupe"); // 실패
```

열거형의 기본 객체는 .enum 속성을 통해 접근할 수 있다.

```ts
FruitEnum.enum.Apple; // "apple"
```

#### Optionals

z.optional()을 사용하면 스키마를 선택적으로 만들 수 있다. 이 함수는 스키마를 ZodOptional 인스턴스로 감싸 결과를 반환한다.

```ts
const schema = z.optional(z.string());
schema.parse(undefined); // => undefined 반환
type A = z.infer<typeof schema>; // string | undefined
```

또한, 기존 스키마에서 .optional() 메서드를 호출할 수 있다.

```ts
const user = z.object({
  username: z.string().optional(),
});
type C = z.infer<typeof user>; // { username?: string | undefined };
```

.unwrap() 메서드를 사용하여 선택적 스키마에서 원래 스키마를 추출할 수 있다.

```ts
const stringSchema = z.string();
const optionalString = stringSchema.optional();
optionalString.unwrap() === stringSchema; // true
```

#### Nullables

z.nullable()을 사용해 null 허용 타입을 만들 수 있다.

```ts
const nullableString = z.nullable(z.string());
nullableString.parse("asdf"); // => "asdf"
nullableString.parse(null); // => null
```

또한, .nullable() 메서드를 사용할 수 있다.

```ts
const E = z.string().nullable();
type E = z.infer<typeof E>; // string | null
```

.unwrap() 메서드를 사용하여 선택적 스키마에서 원래 스키마를 추출할 수 있다.

```ts
const stringSchema = z.string();
const nullableString = stringSchema.nullable();
nullableString.unwrap() === stringSchema; // true
```

#### Objects

```ts
const Dog = z.object({
  name: z.string(),
  age: z.number(),
});

// 추론된 타입을 다음과 같이 추출할 수 있습니다.
type Dog = z.infer<typeof Dog>;

// 다음과 동일합니다:
type Dog = {
  name: string;
  age: number;
};
```

**.shape**
특정 키의 스키마에 접근하려면 .shape를 사용하면 된다.

```ts
Dog.shape.name; // => 문자열 스키마
Dog.shape.age; // => 숫자 스키마
```

**.keyof**
객체 스키마의 키를 기반으로 ZodEnum 스키마를 생성하려면 .keyof를 사용한다.

```ts
const keySchema = Dog.keyof();
keySchema; // ZodEnum<["name", "age"]>
```

**.extend**
.extend 메서드를 사용하여 객체 스키마에 필드를 추가할 수 있다.

```ts
const DogWithBreed = Dog.extend({
  breed: z.string(),
});
```

.extend를 사용하여 필드를 덮어쓸 수도 있으니 주의해서 사용해야 한다.

**.merge**
A.extend(B.shape)과 동일하다

```ts
const BaseTeacher = z.object({ students: z.array(z.string()) });
const HasID = z.object({ id: z.string() });

const Teacher = BaseTeacher.merge(HasID);
type Teacher = z.infer<typeof Teacher>; // => { students: string[], id: string }
```

두 스키마가 동일한 키를 가지고 있으면, B의 속성이 A의 속성을 덮어쓴다.

**.pick** 와 **.omit**
TypeScript의 Pick과 Omit 유틸리티 타입에서 영감을 받아, 모든 Zod 객체 스키마는 .pick 및 .omit 메서드를 사용하여 수정된 버전을 반환한다.

```ts
const Recipe = z.object({
  id: z.string(),
  name: z.string(),
  ingredients: z.array(z.string()),
});

const JustTheName = Recipe.pick({ name: true });
type JustTheName = z.infer<typeof JustTheName>; // { name: string }

const NoIDRecipe = Recipe.omit({ id: true });
type NoIDRecipe = z.infer<typeof NoIDRecipe>; // { name: string, ingredients: string[] }
```

**.partial**
TypeScript의 Partial 유틸리티 타입처럼 .partial 메서드는 모든 속성을 옵셔널로 만든다.

```ts
const user = z.object({
  email: z.string(),
  username: z.string(),
});

const partialUser = user.partial();
// { email?: string | undefined; username?: string | undefined }
```

특정 속성만 선택적으로 만들 수도 있다.

```ts
const optionalEmail = user.partial({
  email: true,
});
// { email?: string | undefined; username: string }
```

**.deepPartial**
.partial 메서드는 얕은 깊이에서만 작동한다. 더 깊은 수준에서 선택적 속성을 만들려면 deepPartial을 사용할 수 있다.

```ts
const user = z.object({
  username: z.string(),
  location: z.object({
    latitude: z.number(),
    longitude: z.number(),
  }),
  strings: z.array(z.object({ value: z.string() })),
});

const deepPartialUser = user.deepPartial();

/*
{
  username?: string | undefined,
  location?: {
    latitude?: number | undefined;
    longitude?: number | undefined;
  } | undefined,
  strings?: { value?: string }[]
}
*/
```

\*deep partials는 객체, 배열 및 튜플의 계층 구조에서만 예상대로 작동한다.

**.required**
.partial 메서드와는 반대로, .required 메서드는 모든 속성을 필수로 만든다.

```ts
const user = z
  .object({
    email: z.string(),
    username: z.string(),
  })
  .partial();
// { email?: string | undefined; username?: string | undefined }

const requiredUser = user.required();
// { email: string; username: string }
```

특정 속성만 필수로 지정할 수도 있다.

```ts
const requiredEmail = user.required({
  email: true,
});
// { email: string; username?: string | undefined }
```

**.passthrough**
기본적으로 Zod 객체 스키마는 파싱 중에 인식되지 않는 키를 제거한다.

```ts
const person = z.object({
  name: z.string(),
});

person.parse({
  name: "bob dylan",
  extraKey: 61, // <- 자동으로 제거됨
});
// => { name: "bob dylan" } // extraKey는 제거됨
```

인식되지 않는 키를 통과시키고 싶다면 .passthrough()를 사용하면 된다.

```ts
person.passthrough().parse({
  name: "bob dylan",
  extraKey: 61,
});
// => { name: "bob dylan", extraKey: 61 }
```

**.strict**
.strict()를 사용하면 인식되지 않는 키를 허용하지 않는다. 입력에 알 수 없는 키가 있으면 오류가 발생한다.

```ts
const person = z
  .object({
    name: z.string(),
  })
  .strict();

person.parse({
  name: "bob dylan",
  extraKey: 61,
});
// => ZodError 발생
```

**.strip**
.strip 메서드를 사용하면 객체 스키마를 기본 동작으로 되돌릴 수 있다. 즉, 인식되지 않는 키를 제거한다.

**.catchall**
.catchall 스키마를 객체 스키마에 전달할 수 있다. 모든 인식되지 않는 키는 해당 스키마에 대해 검증된다.

```ts
const person = z
  .object({
    name: z.string(),
  })
  .catchall(z.number());

person.parse({
  name: "bob dylan",
  validExtraKey: 61, // 정상 동작
});

person.parse({
  name: "bob dylan",
  validExtraKey: false, // 실패
});
// => ZodError 발생
```

.catchall()을 사용하면 .passthrough(), .strip(), .strict()를 사용할 필요가 없이 모든 인식되지 않는 키에 대한 스키마 검증을 할 수 있다.

#### Arrays

```ts
const stringArray = z.array(z.string());

// 동일한 표현
const stringArray = z.string().array();
```

메서드 호출 순서에 주의해야한다.

```ts
z.string().optional().array(); // (string | undefined)[]
z.string().array().optional(); // string[] | undefined
```

**.element**
배열 요소의 스키마에 접근하려면 .element를 사용할 수 있다.

```ts
stringArray.element; // => 문자열 스키마
```

**.nonempty**
배열에 최소 하나의 요소가 있어야 하는 경우 .nonempty()를 사용할 수 있다.

```ts
const nonEmptyStrings = z.string().array().nonempty();
// 추론된 타입: [string, ...string[]]

nonEmptyStrings.parse([]); // 오류 발생: "Array cannot be empty"
nonEmptyStrings.parse(["Ariana Grande"]); // 통과
```

커스텀 에러 메세지를 지정할 수도 있다.

```ts
const nonEmptyStrings = z.string().array().nonempty({
  message: "빈 배열은 허용되지 않습니다!",
});
```

**.min** / **.max** / **.length**

```ts
z.string().array().min(5); // 최소 5개의 항목을 포함해야 함
z.string().array().max(5); // 최대 5개의 항목을 포함해야 함
z.string().array().length(5); // 정확히 5개의 항목을 포함해야 함
```

#### Tuples

배열과 달리, 튜플은 고정된 개수의 요소를 가지며, 각 요소는 다른 타입을 가질 수 있다.

```ts
const athleteSchema = z.tuple([
  z.string(), // 이름
  z.number(), // 등번호
  z.object({
    pointsScored: z.number(), // 통계
  }),
]);

type Athlete = z.infer<typeof athleteSchema>;
// type Athlete = [string, number, { pointsScored: number }]
```

**.rest**
rest 인자는 .rest 메서드를 사용해 추가할 수 있다.

```ts
const variadicTuple = z.tuple([z.string()]).rest(z.number());
const result = variadicTuple.parse(["hello", 1, 2, 3]);
// => [string, ...number[]];
```

#### Unions

Zod는 OR 타입ㅇ르 구성하기 위한 내정 메서드인 z.union을 포함하고 있다.

```ts
const stringOrNumber = z.union([z.string(), z.number()]);

stringOrNumber.parse("foo"); // 통과
stringOrNumber.parse(14); // 통과
```

**.or**
.or 메서드를 사용해 더 간단하게도 사용 가능하다.

```ts
const stringOrNumber = z.string().or(z.number());
```

#### Discriminated Unions

Discriminated Unions은 특정 키를 공유하는 객체 스키마들의 유니언이다.

```ts
const myUnion = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("failed"), error: z.instanceof(Error) }),
]);

myUnion.parse({ status: "success", data: "yippie ki yay" });
```

#### Records

Records 스키마는 Records<string, number>와 같은 타입을 검증하는 데 사용된다.

```ts
const User = z.object({ name: z.string() });
const UserStore = z.record(z.string(), User);

type UserStore = z.infer<typeof UserStore>;
// => Record<string, { name: string }>
```

#### Maps

```ts
const stringNumberMap = z.map(z.string(), z.number());
type StringNumberMap = z.infer<typeof stringNumberMap>;
// type StringNumberMap = Map<string, number>
```

#### Sets

```ts
const numberSet = z.set(z.number());
type NumberSet = z.infer<typeof numberSet>;
// type NumberSet = Set<number>
```

Sets 스키마는 다음과 같은 유틸리티 메서드로 추가 제약을 걸수있다. // 배열과 비슷하네?

```ts
z.set(z.string()).nonempty(); // 최소 하나의 항목이 있어야 함
z.set(z.string()).min(5); // 최소 5개의 항목이 있어야 함
z.set(z.string()).max(5); // 최대 5개의 항목이 있어야 함
z.set(z.string()).size(5); // 정확히 5개의 항목이 있어야 함
```

#### Intersections

Intersections는 두 객체 타입을 교차하여 "AND" 타입을 만드는 데 유용하다

```ts
const Person = z.object({
  name: z.string(),
});

const Employee = z.object({
  role: z.string(),
});

const EmployedPerson = z.intersection(Person, Employee);
// => { name: string; role: string }
```

Intersections 보다는 .merge()를 사용하는 것을 추천한다.
