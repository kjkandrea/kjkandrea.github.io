---
title: "react-hook-form 으로 React 제출 양식(form) 에서 재사용성 찾기"
date: 2023-01-17 12:00:00 +0900
categories: frontend
---

제출 양식(form)은 사용자와 서버가 상호작용 할 수 있는 대화형 개체이다. 
많은 정보 입력이 필요한 쇼핑몰의 관리자 페이지나 채용 플랫폼 같은 서비스에서의 제출 양식은 수많은 필드가 존재한다. 
더군다나 사용자가 작성한 양식이 서버에 도달하기 이전에 검증(validation) 로직을 브라우저에 컨트롤한다면 더욱 복잡도가 높아진다.

이러한 제출 양식이 어떠한 일을 하는지 한번 나열해보자.

1. 입력 가능한 필드를 제공한다.
2. 입력이 완료되면 서버에 전송한다. 
3. (서버 측 검증) 서버에서 400 번대 에러가 응답되었다면, 사용자에게 피드백을 제공한다. 
4. (브라우저 측 검증) 필수 입력 필드를 입력하지 않았다면/입력 양식이 정합하지 않다면, 사용자에게 피드백을 제공한다.
5. 사용자가 입력 필드를 작성하다가 제출하지 않은 채 이탈하려 한다면, 작성중이던 양식이 사라질 수 있다는 걸 확인시킨다.
6. 입력 양식을 이용하여 검색을 한다면 search params 와 사용자가 입력한 양식을 동기화한다.

4, 5 번은 상황에 따라 구현이 필요한 불필요한 경우도 있지만, 1, 2, 3, 4 번은 제출 양식을 구현할때에 대부분 등장한다.

## 구현 해보기

1, 2, 3, 4 번의 기능을 관심사의 분리가 이루어지지않은 컴포넌트로 구현해 복잡성을 체감해볼까?

```tsx
import { signInMutation } from 'store/auth'

function SignIn() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [clientErrorMessage, setClientErrorMessage] = useState('');

  const {
    mutate,
    // 3. (서버 측 검증) 서버에서 400 번대 에러가 응답되었다면, 사용자에게 피드백을 제공한다. 
    error: {message: serverErrorMessage},
  } = signInMutation();

    // 4. (브라우저 측 검증) 필수 입력 필드를 입력하지 않았다면/입력 양식이 정합하지 않다면, 사용자에게 피드백을 제공한다.
  const validation = (): boolean => {
    if (!email.trim()) {
      setClientErrorMessage('이메일을 입력하세요.');
      return false;
    }

    if (!password.trim()) {
      setClientErrorMessage('비밀번호를 입력하세요.');
      return false;
    }
      
    return true;
  }

  // 2. 입력이 완료되면 서버에 전송한다. 
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
      
    const invalid = !validation();
    if(invalid) return;

    mutate({
      email,
      password,
    });
  };

  // 1. 입력 가능한 필드를 제공한다.
  return (
    <form onSubmit={e => handleSubmit(e)}>
      <input
        type="text"
        name="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
      />
      <input
        type="password"
        name="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
      />
      {clientErrorMessage}
      {serverErrorMessage}
      <input type="submit" value="로그인" />
    </form>
  );
}
```

가급적 단순화하고자 노력했음에도 60 줄이 약간 넘는 코드가 되었다. 여러 관심사가 하나의 컴포넌트로 응집되어 있는 모습이다. 우선 재사용성 관점에서 생각해보자.

email, password 필드가 EmailField, PasswordField 컴포넌트로 정의된다면 어떨까? `*Field` 컴포넌트가 맡게 될 관심사는 다음과 같다.

* 1\. 입력 가능한 필드를 제공한다.
* 4\. (브라우저 측 검증) 필수 입력 필드를 입력하지 않았다면/입력 양식이 정합하지 않다면, 사용자에게 피드백을 제공한다.

두가지 관심사를 서브루틴에서 처리하도록 컴포넌트를 만들어보자.

```tsx
interface FieldProps {
  onComplete: (value: string) => void;
}

export default function EmailField({onComplete}: FieldProps) {
  const [value, setValue] = useState('');
  const [clientErrorMessage, setClientErrorMessage] = useState('');

  const validation = (email: string): boolean => {
    if (!email.trim()) {
      setClientErrorMessage('이메일을 입력하세요.');
      return false;
    }

    return true;
  };

    const tryOnComplete = () => validation(value) ? onComplete(value) : onComplete('');

  return (
    <>
      <input
        type="email"
        name="email"
        value={value}
        onChange={e => setValue(e.target.value)}
        onBlur={() => tryOnComplete()}
      />
      {clientErrorMessage}
    </>
  );
}

function PasswordField({
  onComplete,
}: FieldProps) {
  const [value, setValue] = useState('');
  const [clientErrorMessage, setClientErrorMessage] = useState('');

  const validation = (email: string): boolean => {
    if (!email.trim()) {
      setClientErrorMessage('패스워드를 입력하세요.');
      return false;
    }

    return true;
  };

  const tryOnComplete = () => validation(value) ? onComplete(value) : onComplete('');

  return (
    <>
      <input
        type="password"
        name="password"
        value={value}
        onChange={e => setValue(e.target.value)}
        onBlur={() => tryOnComplete()}
      />
      {clientErrorMessage}
    </>
  );
}
```
```tsx
function SignIn() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const {
    mutate,
    error: {message: serverErrorMessage},
  } = signInMutation();

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    mutate({
      email,
      password,
    });
  };

  return (
    <form onSubmit={e => handleSubmit(e)}>
      <EmailField onComplete={email => setEmail(email)} />
      <PasswordField onComplete={password => setPassword(password)} />
      {serverErrorMessage}
      <input type="submit" disabled={!(email && password)} value="로그인" />
    </form>
  );
}
```

관심사의 분리 측면에서는 나아졌으나, `*Field` 컴포넌트를 생성하는 규칙이 필요할것으로 보인다. 더군다나 검증 로직을 무리하게 서브 컴포넌트로 이양한 탓에 각 필드가 검증되는 시점이 모호해졌다. 이를 해소하고자 Submit 버튼에 조건부 disabled 를 부여했으나 사용자 친화적인 구현으로 보기엔 부족한 부분이 많다.

정리하자면 위 버전에서는 다음과 같은 부분을 보완해야한다.

* `*Field` 의 생성 규칙을 정의한다.
  * 추가적인 계층을 두어 중복을 최소화 한다. 
  * 검증 규칙을 일원화 할 수 있는 설계가 필요하다.
* Submit 시점에 각 필드 검증이 되어야한다. 
* 설계를 개선하기위해 UX 가 나빠져서는 안된다.

위 사항들을 보완한다면 `*Field` 컴포넌트를 SignIn 이외에 비밀번호 찾기, 비밀번호 변경, 회원가입 등에서도 재사용 할 수 있을것이다. 

## react-hook-form

부모 컴포넌트에서의 submit 시점을 알 수 있도록 추가적인 prop 을 내리게 하고, 
Field 의 구현 규칙이 정의된 커스텀 훅을 만든다거나 하는 식으로 시도해 볼 수 있겠다.





## reference

* [Create a Reusable Text Input With React Hook Form](https://blog.qoddi.com/create-a-reusable-text-input-with-react-hook-form/)
