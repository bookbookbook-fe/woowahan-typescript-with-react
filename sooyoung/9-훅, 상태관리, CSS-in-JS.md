# 훅

## useState

- 상태 관리를 위한 훅으로 튜플 형태를 반환함
- 타입스크립트를 사용하면 예상치 못한 사이드 이펙트를 방지할 수 있음
- 제네릭 타입 선언으로 타입 안정성을 확보함

```typescript
function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];

type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S);
```

## 의존성 배열을 사용하는 훅

### useEffect와 useLayoutEffect

- useEffect: 렌더링 이후에 어떤 일을 수행해야 하는지 알려주는 훅
- promise 타입은 반환하지 않으므로 useEffect의 콜백함수에는 비동기 함수가 들어갈 수 없음
- deps에 객체나 배열을 넣을 때는 얕은 비교로 판단하므로 주의해야 함
- deps가 빈 배열이면 컴포넌트가 처음 렌더링될 때만 실행됨

```typescript
function useEffect(effect: EffectCallback, deps?: DependencyList): void;
type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

- useLayoutEffect: 화면에 컴포넌트가 그려지기 전에 콜백 함수를 실행하여 첫번째 렌더링때 빈 상태값이 뜨는 경우를 방지할 수 있음

### useMemo와 useCallback

- 이전에 생성된 값 또는 함수를 기억하는 훅
- 어떤 값을 계산하는데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 유용
- 과도한 메모이제이션은 성능 저하를 가져올 수 있으므로 적절히 사용해야 함

```typescript
function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: DependencyList
): T;
```

## useRef

- DOM을 직접 선택해야할 때 사용(input focus 등)
- useRef는 세 종류의 타입 정의를 가짐
- 넘겨주는 인자 타입에 따라 반환되는 타입이 달라짐
- useRef로 관리되는 변수는 값이 바뀌어도 리렌더링이 발생하지 않음

```typescript
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;
```

- HTMLInputElement만 넘겨주면 RefObject 타입으로 반환되어 ref.current는 변경할 수 없음
- HTMLInputElement | null을 넘겨주면 MutableRefObject 타입이 반환되어 ref.current는 변경할 수 있음

### 자식 컴포넌트에 ref 전달하기

- forwardRef를 사용하여 ref를 prop으로 전달할 수 있음
- useImperativeHandle 훅으로 부모 컴포넌트에서 자식 컴포넌트의 메서드를 호출할 수 있게 함

# 상태관리

## 상태란?

- 상태란 시간이 지나면서 변할 수 있는 동적인 데이터
- 값이 변경될 때마다 컴포넌트의 렌더링 결과물에 영향을 줌
- 지역 상태, 전역 상태, 서버 상태로 분류 가능

## 상태의 종류

### 지역 상태

- 컴포넌트 내부에서 관리되는 상태
- 주로 useState 훅, useReducer 훅에 의해 관리됨

### 전역 상태

- 앱 전체에서 공유하는 상태
- 여러 컴포넌트가 사용할 수 있음
- Prop drilling 문제를 피할 때 지역 상태를 전역 상태로 공유 가능

### 서버 상태

- 외부 서버에 저장해야 하는 상태들
- UI 상태와 결합하여 관리
- 로딩 여부나 에러 상태 등을 포함
- react-query, SWR 등의 라이브러리로 관리 가능

## 상태를 잘 관리하기 위한 가이드

### 시간이 지나도 변하지 않는다면 상태가 아니다

- 객체 참조 동일성 유지 방법으로 다음 세 가지를 고려할 수 있음:
  - useMemo: 성능 향상을 위한 용도로만 사용해야 함
  - useState: 의미론적으로는 적절하지 않음
  - useRef: 공식 문서에서 권장하는 방법

### 파생된 값은 상태가 아니다

- SSOT(Single Source Of Truth) 원칙: 데이터는 단 하나의 출처에서 생성/수정되어야 함
- props나 기존 상태에서 계산될 수 있는 값은 상태가 아님
- 계산 비용이 클 경우 useMemo로 최적화 가능

## 전역 상태 관리 방법

### Context API

- 컴포넌트 간 데이터 공유를 위한 API
- Provider와 useContext 훅으로 구성
- 대규모 애플리케이션에서는 성능 이슈가 발생할 수 있음

### 상태 관리 라이브러리

#### Mobx

- 객체 지향 및 반응형 프로그래밍 패러다임 기반
- 상태 변경 로직을 단순하게 작성 가능
- 데이터 변화 추적이 어려울 수 있음

#### Redux

- 함수형 프로그래밍 영향을 받은 라이브러리
- UI 프레임워크에 종속되지 않음
- 보일러플레이트가 많고 사용 난이도가 높음

#### Recoil

- Atom과 selector를 통해 상태 관리
- 보일러플레이트가 적고 러닝커브가 낮음
- 실험적인 상태로 다양한 요구사항 검증이 이루어지지 않음

#### Zustand

- Flux 패턴을 사용하는 상태관리 라이브러리
- 훅 기반의 편리한 API 모듈 제공
- 커뮤니티와 생태계가 작은 편

# Chapter 3. CSS-in-JS

## CSS-in-JS의 개념

- 자바스크립트로 스타일을 선언적이고 유지보수할 수 있는 방식으로 표현
- 인라인 스타일은 DOM 노드에 속성으로 스타일 추가, CSS-in-JS는 DOM 상단에 스타일 추가
- 미디어 쿼리, 슈도 선택자 등 CSS 기능을 사용 가능

## CSS-in-JS의 장점

1. 컴포넌트 단위로 스타일 추상화 가능
2. 부모 요소에서 자동으로 상속되는 속성에서 독립적
3. 고유한 스코프를 가져 선택자 충돌 방지
4. 자동 벤더 프리픽스 추가로 브라우저 호환성 향상
5. 자바스크립트 변수/함수를 스타일 코드에서 쉽게 사용 가능

## 관련 개념

### BEM(Block Element Modifier)

- CSS 클래스 네이밍 컨벤션
- 블록(컴포넌트), 요소(하위 요소), 수정자(상태/특성)로 구성

### 벤더 프리픽스(Vendor Prefix)

- 브라우저마다 지원되는 CSS 속성을 위한 접두사
- 브라우저 호환성을 위해 사용

## 등장 배경

- CSS의 7가지 문제점 해결을 위해 등장:
  1. 글로벌 네임스페이스
  2. 의존성 관리 문제
  3. 불필요한 코드 제거 어려움
  4. 클래스 이름 최소화 어려움
  5. 자바스크립트와 상수 공유 어려움
  6. CSS 로드 순서에 따른 우선순위 문제
  7. 외부 수정 관리 어려움

## TypeScript와 함께 사용하기

- styled-components에서 반복되는 타입 선언 문제 발생
- Pick, Omit 같은 유틸리티 타입을 활용해 중복 타입 선언 피하기

# 비동기 호출

## API 요청

### fetch로 API로 요청하기

- 동일한 API URL을 여러군데에서 사용하게 되면 복붙하여 사용하게 된다.
- 컴포넌트에 깊숙이 자리잡은 비동기 호출코드는 백엔드등에서 기능 변경 요구에 취약하다.

### 서비스 레이어로 분리하기

- 여러 API 요청 정책이 추가되어 코드가 변경 될 수 있다는 것을 감안한다면, 비동기 호출 코드는 컴포넌트 영역에서 분리되어 서비스 레이어에서 처리되어야 한다.

### Axios 활용하기

### Axios 인터셉터 사용하기

- Axios의 인터셉터를 사용하면 에러등을 처리할 때 하나의 에러 객체로 묶어서 처리할 수 있다.

### API 응답 타입 지정하기

- API응답 값은 하나의 Response 타입으로 묶일 수 있다.
- UPDATE나 CREATE같이 응답이 없을 수 있는 API를 처리하기 까다로워진다.
- 따라서 Response 타입은 apiRequester가 모르게 관리되어야 한다.

### 뷰 모델(View Model) 사용하기

- API 응답은 변할 가능성이 크다
- 특히 새로운 프로젝트는 서버 스펙이 자주 바뀌기 떄문에 뷰모델을 사용하여 API 변경에 따른 범위를 한정해줘야 한다.
- 흔히 좋은 컴포넌트는 병견될 이유가 하나뿐인 컴포넌트라고 한다.
- 뷰 모델을 만들면 APPI 응당입 바뀌어도 UI가 깨지지 않게 개발할 수 있다.
- 그러나 뷰 모델 방식은 추상화 레이어 추가는 코드를 복잡하게 하고 레이어를 관리하고 개발하는데 비용이 많이든다.

### Superstruct를 사용해 런타임에서 응답 타입 검증하기

## API 상태 관리하기

- 상태 관리 라이브러리로 호출하기
- 훅으로 호출하기

## API 에러 핸들링

### 타입가드 활용하기

- Axios에서 타입가드를 사용할 수 있다.

### 에러 서브클래싱하기

- 에러 서브클래싱이란 기존(상위 또는 부모) 클래스를 확장하여 새로운(하위 또는 자식) 클래스를 만드는 과정을 말한다.
- 새로운 클래스는 상위 클래스의 모든 속성과 메서드를 상속받아 사용할 수 있고 추가적인 속성과 메서드를 정의할 수도 있다.

### 인터셉터를 활용한 에러처리

- Axios 같은 펮칭 라이브러리는 인터셉터 기능을 제공한다.

### 에러 바운더리를 활용한 에러 처리

- 에러 바운더리는 리액트 컴포넌트 트리에서 에러가 발생할 때 공통으로 에러리르 처리하는 리액트 컴포넌트이다.
- 에러 바운더리를 사용하면 리액트 컴포넌트 트리 하위에 있는 컴포넌트에서 발생한 에러를 캐치하고, 해당 에러를 가장 가까운 부모 에러 바운더리에서 처리하게 할 수 있다.

### 상태 관리 라이브리에서의 에러처리

### react-query를 활용한 에러처리

### 그 밖의 에러처리

- if 문을 통해 data.status를 확인하여 처리한ㄷ.

## API 모킹

- 모킹은 가짜 모듈을 활용하는 것을 말한다.
- 서버 상태에 문제가 발생한 경우에도 서버의 영향을 받지 않고 프론트엔드 개발을 할 수 있게 된다.
- JSON 파일 불러오기
- NextApiHandler 활용하기
- API 요청 핸들러에 분기 추가하기
- axios-mock-adapter로 모킹하기
- 목업 사용 여부 제어하기
