# JSX에서 TSX로: 간략 가이드

## 리액트 컴포넌트 타입

### 함수 컴포넌트 타입 지정하기

```typescript
// 방법 1: 함수 선언 방식
function Welcome(props: WelcomeProps): JSX.Element {}

// 방법 2: React.FC 사용
const Welcome: React.FC<WelcomeProps> = ({ name }) => {};

// 방법 3: 직접 타입 지정
const Welcome = ({ name }: WelcomeProps): JSX.Element => {};
```

> **참고**: React v18부터는 FC에서 children이 제거되었으며, VFC는 deprecated 되었습니다.

### children 타입 지정

```typescript
// 기본 방식
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };

// 특정 값으로 제한하기
type WelcomeProps = {
  name: string;
  children: '귀한 분' | '고마운 분';
};
```

### 컴포넌트 반환 타입

- **ReactElement**: 구체적인 props 타입을 지정할 수 있음
- **JSX.Element**: ReactElement를 확장한 타입, props가 any로 지정됨
- **ReactNode**: ReactElement 외에도 string, number, boolean 등 포함

```typescript
// ReactNode 사용 예시 (children 타입)
interface MyComponentProps {
  children?: React.ReactNode;
}

// JSX.Element 사용 예시 (컴포넌트 prop)
interface Props {
  icon: JSX.Element;
}

// ReactElement 사용 예시 (props 타입 추론)
interface Props {
  icon: React.ReactElement<IconProps>;
}
```

## JSX를 TSX로 변환하기

### 원본 JSX 컴포넌트

```javascript
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = (e) => {
    const selected = Object.entries(options).find(
      ([, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <select
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  );
};
```

### 1. 기본 타입 정의

```typescript
type Option = Record<string, string>;

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void;
}

const Select = ({
  options,
  selectedOption,
  onChange,
}: SelectProps): JSX.Element => {
  // ... 구현 ...
};
```

### 2. 이벤트 핸들러 타입 지정

```typescript
const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
  const selected = Object.entries(options).find(
    ([, value]) => value === e.target.value
  )?.[0];
  onChange?.(selected);
};
```

### 3. 제네릭 컴포넌트로 개선

```typescript
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  options,
  selectedOption,
  onChange,
}: SelectProps<OptionType>): JSX.Element => {
  // ... 구현 ...
};
```

### 4. HTML 속성 추가

```typescript
type ReactSelectProps = React.ComponentPropsWithoutRef<'select'>;

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps['id'];
  className?: ReactSelectProps['className'];
  // ... 기존 props ...
}
```

### 5. 스타일 속성 추가

```typescript
// 테마 정의
export const theme = {
  fontSize: { default: '16px', small: '14px', large: '18px' },
  color: { white: '#FFFFFF', black: '#000000' },
};

type Theme = typeof theme;
export type FontSize = keyof Theme['fontSize'];
export type Color = keyof Theme['color'];

// 스타일 props
interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

// SelectProps에 스타일 속성 추가
interface SelectProps<OptionType extends Record<string, string>>
  extends Partial<SelectStyleProps> {
  // ... 기존 props ...
}
```

### 완성된 컴포넌트 사용 예시

```typescript
const fruits = {
  apple: '사과',
  banana: '바나나',
  blueberry: '블루베리',
};
type Fruit = keyof typeof fruits;

export const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select
      className="fruitSelectBox"
      options={fruits}
      onChange={changeFruit}
      selectedOption={fruit}
      fontSize="large"
    />
  );
};
```

## 타입 안전성을 위한 팁

1. **타입 범위를 좁게 유지하기**: `string` 대신 `'apple' | 'banana'` 같이 구체적인 유니온 타입 사용
2. **제네릭 활용하기**: 컴포넌트에 입력되는 데이터 구조에 따라 타입이 유연하게 조정되도록 설계
3. **HTML 속성 재활용하기**: `React.ComponentPropsWithoutRef`를 활용하여 기본 HTML 속성 재사용
4. **이벤트 핸들러에 올바른 타입 적용하기**: 리액트의 SyntheticEvent 타입 사용
