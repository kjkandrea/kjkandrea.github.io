---
title: "React 제출 양식에서 입력 필드의 재사용성 찾기"
date: 2023-01-18 12:00:00 +0900
categories: frontend
---

제출 양식(form)은 사용자와 서버가 상호작용 할 수 있는 대화형 개체이다. 
많은 정보 입력이 필요한 쇼핑몰의 관리자 페이지나 채용 플랫폼 같은 서비스에서의 제출 양식은 수많은 필드가 존재한다.

작성된 양식이 서버에 도달한 후 거부되면 사용자에게 지시하기 이전에 
잘못된 입력에 대한 정보를 보다 이른 시점에 제공하고자 유효성 검증(validate) 로직을 브라우저에서 컨트롤하곤 한다. 

이러한 제출 양식이 어떠한 일을 하는지 한번 나열해보자.

1. 입력 가능한 필드를 제공한다.
2. 입력이 완료되면 서버에 전송한다. 
3. (서버 처리 결과 알림) 서버에서 400 번대 에러가 응답되었다면, 사용자에게 피드백을 제공한다. 
4. (브라우저 측 검증) 필수 입력 필드를 입력하지 않았다면/입력 양식이 정합하지 않다면, 사용자에게 피드백을 제공한다.
5. 사용자가 입력 필드를 작성하다가 제출하지 않은 채 이탈하려 한다면, 작성중이던 양식이 사라질 수 있다는 걸 확인시킨다.
6. 입력 양식을 이용하여 검색을 한다면 search params 와 사용자가 입력한 양식을 동기화한다.

5, 6 은 상황에 따라 구현이 필요한 불필요한 경우도 있지만, 1, 2, 3, 4 은 제출 양식을 구현할때에 대부분의 경우 필수적이다.

## 제출 양식을 통한 사용자 인증 을 구현해보자.

1, 2, 3, 4 의 기능을 무작정 구현하여 복잡성을 체감해볼까?

```tsx
import { signInMutation } from 'store/auth'

function SignIn() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [clientErrorMessage, setClientErrorMessage] = useState('');

  const {
    mutate,
    // 3. (서버 처리 결과 알림) 서버에서 400 번대 에러가 응답되었다면, 사용자에게 피드백을 제공한다. 
    error: {message: serverErrorMessage},
  } = signInMutation();
  
  // 4. (브라우저 측 검증) 필수 입력 필드를 입력하지 않았다면/입력 양식이 정합하지 않다면, 사용자에게 피드백을 제공한다.
  const validate = (): boolean => {
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
      
    const invalid = !validate();
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

여러 관심사가 하나의 컴포넌트로 응집되어 있는 모습이다. 재사용성 관점에서 생각해보자.
이메일, 패스워드 필드가 이 양식에서만 등장할까? 이메일은 회원가입, 이메일을 통해 비밀번호 찾기 양식 등에서 등장할 수 있고 마찬가지로 비밀번호 필드도 여러 양식에서 등장할 수 있다.

각 필드들이 EmailField, PasswordField 컴포넌트로 정의된다면 어떨까? `*Field` 컴포넌트가 맡게 될 관심사는 다음과 같다.

* 1\. 입력 가능한 필드를 제공한다.
* 4\. (브라우저 측 검증) 필수 입력 필드를 입력하지 않았다면/입력 양식이 정합하지 않다면, 사용자에게 피드백을 제공한다.

구현 중점으로 조금 더 명확하게 정의해보자면 다음과 같다.

* 제출 이벤트가 일어날 때에 부모로 부터 메시지를 받을 수 있다.
* 입력 가능한 Input 을 제공한다.
* 입력된 값을 부모에게 전달할 수 있다.
* 유효성 검증 규칙을 내부에서 지니고 있다.

### 리액트의 설계원칙대로 부모에서 자식으로 메시지 전달하기

`*Field` 를 사용하는 부모 컴포넌트에서 `*Field` 에게 제출되는 시점에 유효성 검증을 요청해야한다. 

리액트 컴포넌트가 아닌 객체라고 가정하면 부모 객체에서 자식 객체로 메시지를 보내는것은 비교적 간단하다.
제출이 일어나는 시점에 `this.emailField.validate()` 자식 객체에서 지닌 메서드를 다음과 같이 호출하면 될것이다.

리액트는 이러한 부모가 자식을 제어하는 구조를 권장하지않고, 예상 가능한 데이터의 흐름과 구조의 단순화를 위해 단방향 데이터 흐름을 통해 컴포넌트를 설계하기를 권장한다.

그렇다면 조금 다르게 사고해보자. 제출이 일어나는 특정 시점에 메서드등 자식이 내부에서 지니고 있는 기능을 호출하는 것이 아닌, 특정 시점에 호출될 메시지를 **자식에서 사전에 정의하여 부모에게 전달해둔다면 어떨까?**

위 정보를 토대로 `FieldRegister` 를 구성해보자.

```tsx
// 자식 컴포넌트에서 *Field 컴포넌트를 정의하려면 fieldRegister 를 거쳐야한다. 
export type FieldRegister = (
  // field 명, 유효성 검증 규칙을 fieldRegister 를 통해 전달해야한다.
  fieldName: string,
  validate: (value: string) => boolean
) => {
  name: string
};

const fieldRegister: FieldRegister = (fieldName, validate) => {
  // ...
  return {
    name: fieldName
  }
}

// ...
<EmailField fieldRegister={fieldRegister} />
```

FieldRegister 를 통해 이메일 필드를 생성하는 EmailField 컴포넌트도 러프하게 정의해볼까?

예제에서는 적당한 복잡성을 유지하기위해 입력값이 있는지만을 검증하겠지만, 앞으로 정의될 `validate` 의 검증 규칙이 정규표현식이 들어가고 길이 제한이 있는 등 매우 디테일하다고 상상해보자.

```tsx
function EmailField({ fieldRegister }: { fieldRegister: FieldRegister }) {
  // 유효성 검증 실패시 보여줄 안내 문구 state
  const [guideMessage, setGuideMessage] = useState('');

  return (
    <div class="field">
      <input
        type="email"
        // 📌 FieldRegister 의 규칙에 따라 fieldName, validate 를 정의한다.
        {...fieldRegister('email', (value: string) => {
          if (!value.trim()) {
            setGuideMessage('이메일을 입력하세요.');
            return false;
          }

          return true;
        })}
      />
      {guideMessage}
    </>
  );
}
```

주석의 📌 부분은 리액트의 디자인 패턴인 [Props Getter Pattern](https://simsimjae.medium.com/react-design-pattern-props-getter-pattern-5d3cf6f0b495) 패턴을 따른다. register 의 리턴값이 `<input>` 에 바인딩 됨으로
fieldRegister 의 리턴값을 통해 부모에서 인풋을 제어할 수 있다. 가령 onChange 를 리턴한다면 실시간으로 값을 부모에서 조회할 수 있는 구조를 갖출 수 있을것이다.

### 제출이 일어나는 시점에 유효성 검증 실행

부모에서 validate 를 자식에게 전달받을 수 있음으로, 부모에서 원하는 시점에 유효성 검증을 실행할 수 있다.

제출(submit) 과 fieldRegister 규칙을 커스텀 훅을 통해 캡슐화할 수 있도록 만들어보자. 커스텀 훅의 이름은 `useForm` 이다.

```tsx
interface FormField {
  validate: (value: string) => boolean;
  value: string;
}

function useForm<FormValues>() {
  const formFieldsRef = useRef<{
    // 딕셔너리 구조로 각 필드의 validate, value 를 관리한다.
    [key: string]: FormField;
  }>({});

  const register: FieldRegister = (fieldName, validate) => {
    const field = {
      validate,
      value: '',
    };
    // formFieldsRef 에 register 를 통해 생성된 필드를 등록한다.
    formFieldsRef.current = {
      ...formFieldsRef.current,
      [fieldName]: field,
    };

    return {
      name: fieldName,
      onChange: e => {
        field.value = e.target.value;
      },
    };
  };
 
  // 훅에서 제출 이벤트 핸들러를 제공한다.
  const handleSubmit = (
    e: React.FormEvent<HTMLFormElement>,
    // 양식 검증이 완료되었다면 next 콜백을 호출한다.
    next: (result: FormValues) => void
  ) => {
    e.preventDefault();

    // formFieldsRef 에 등록된 모든 양식이 validate: true 라면 검증된(true) 값이다.  
    const valid = Object.values(formFieldsRef.current).every(
      ({validate, value}) => validate(value)
    );

    if (!valid) return;

    const formValues = Object.entries(formFieldsRef.current).map(
      ([fieldName, {value}]) => [fieldName, value]
    );

    // 검증이 성공하였을 경우 next 를 호출한다.
    next(Object.fromEntries(formValues));
  };

  return {
    register,
    handleSubmit,
  };
}
```

자 이제 준비된 useForm 커스텀 훅을 부모 컴포넌트에서 사용해보자.\
[react-hook-form](https://react-hook-form.com) 라이브러리를 사용해보았다면 react-hook-form 의 `useForm` 과 유사한 구조를 지님을 알아차렸을 수도 있겠다.

```tsx
function SignIn() {
  const {handleSubmit, register} = useForm<{
    email: string;
    password: string;
  }>();

  const {
    mutate,
    error: {message: serverErrorMessage},
  } = signInMutation();

  return (
    <form onSubmit={e => handleSubmit(e, mutate)} noValidate>
      <EmailField register={register} />
      <PasswordField register={register} />
      {serverErrorMessage}
      <input type="submit" value="로그인" />
    </form>
  );
}
```

무엇이 나아졌는가? 가장 눈에 띄는 부분은 부모 컴포넌트 SignIn 과 자식 컴포넌트 `*Field` 가 각자의 관심사를 나눠가졌다는 것이다.

**부모 컴포넌트 SignIn**

* 입력이 완료되면 서버에 전송한다.
* (서버 처리 결과 알림) 서버에서 400 번대 에러가 응답되었다면, 사용자에게 피드백을 제공한다.

**자식 컴포넌트 EmailField, PasswordField**

* 입력 가능한 필드를 제공한다.
* (브라우저 측 검증) 필수 입력 필드를 입력하지 않았다면/입력 양식이 정합하지 않다면, 사용자에게 피드백을 제공한다.

**커스텀 훅 useForm**

* 부모, 자식이 메시지를 주고받는 방식을 제공한다.
* Field 의 생성 규칙을 정의한다.

또한 자식 컴포넌트는 재사용이 가능하다. 이 글에서는 입력요소(`<input>`) 만을 다루었지만 입력요소 조합된 형태의 fieldSet 이나, 체크 박스, 선택 입력 등을 Field 계층과 동일한 접근법을 통해 정의할 수 있음을 예상할 수 있다.

Field 의 관심사가 명확해졌고, 그에 따라 정의된 EmailField, PasswordField 컴포넌트는 회원가입, 이메일을 통해 비밀번호 찾기, 비밀번호 변경 등에 범용적으로 사용될 수 있다.

## react-hook-form 소개

[react-hook-form](https://react-hook-form.com/api/useform) 은 위의 예제와 유사한 방식으로 동작하는 리액트 폼 라이브러리이다.

사실 'React 제출 양식에서 입력 필드의 재사용성 찾기' 의 방안을 찾아보던 중 `react-hook-form` 을 먼저 접하게 되었고, 위 코드는 해당 라이브러리에서 [`useForm`](https://react-hook-form.com/api/useform), [`register`](https://react-hook-form.com/api/useform/register) 의 설계를 이해하기 위해 직접 구현해본 것이다.

유효성 검증 시점을 선택할 수 있다거나, 구체적인 validate 규칙을 `pattern` 객체를 통해 제공하는 등 uncontrolled 하게 form 을 관리하기 위한 많은 부가기능과 여러 룰을 제공한다. 

팀 내의 특정한 규칙이 없다면 입력 양식 관리는 개인 스타일에 따라 중구난방이 되기 마련인데, 
`react-hook-form` 은 팀 단위로 일하는 클라이언트 개발 조직에서 규칙성있게 제출 양식을 관리하는 합리적인 대안으로 보인다.

react-hook-form 으로 구현된 로그인 양식은 다음과 같다. 아래 예시에서는 register 를 prop 으로 전달하는 방식대신 [useFormContext](https://react-hook-form.com/api/useformcontext) 훅을 사용하여 부모와 메시지를 주고받도록 하였다. 

```tsx
import {SubmitHandler, useForm, FormProvider} from 'react-hook-form';

export default function SignInPage() {
  const methods = useForm<SignInFormValues>({
    mode: 'onSubmit',
  });

  const {
    mutate,
    error: {message: serverErrorMessage},
  } = signInMutation();

  return (
    <FormProvider {...methods}>
      <Form onSubmit={methods.handleSubmit(mutate)} noValidate>
        <EmailField />
        <PasswordField />
        <button type="submit">로그인</button>
      </Form>
    </FormProvider>
  )
}
```

```tsx
import {useFormContext} from 'react-hook-form';

function EmailField() {
  const {
    register,
    formState: {errors},
  } = useFormContext<EmailValues>();

  return (
    <Form.Input
      id="email"
      type="email"
      label="email"
      {...register('email', {
        required: '이메일을 입력하세요.',
      })}
      helperText={errors.email?.message}
    />
  );
}
```

## 마치며

예전에 경험했던 프로젝트에서 입력 양식을 규칙 없이 관리하여 수십개의 동일한 브랜드 검색 기능을 지니는 폼을 일괄 수정해야할 때 터무니없는 오랜 시간이 걸려 팀의 사기가 저하되는 경험을 한적이 있다.

이렇게라도 정리해두니 동일한 시행착오를 밞지 않을것같아 한시름 놓인다.

특정 라이브러리의 최소 기능 버전을 러프하게 구현해보는 것은 해당 라이브러리를 이해하기에 많은 도움이 되는것같다.

## reference

* [react-hook-form](https://react-hook-form.com/api/useform)
* [Qoddi : Create a Reusable Text Input With React Hook Form](https://blog.qoddi.com/create-a-reusable-text-input-with-react-hook-form/)
* [심재철 : \[React Design Pattern\] Props Getter Pattern](https://simsimjae.medium.com/react-design-pattern-props-getter-pattern-5d3cf6f0b495)
