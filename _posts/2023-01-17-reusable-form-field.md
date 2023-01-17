---
title: "React 제출 양식(form) 에서 재사용성 찾기"
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
    <form onSubmit={e => console.log(e)}>
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
      <button type="submit">로그인</button>
    </form>
  );
}
```

가급적 단순화하고자 노력했음에도 60 줄이 약간 넘는 코드가 되었다. 여러 관심사가 하나의 컴포넌트로 응집되어 있지만 우선 재사용성 관점에서 생각해보자.
