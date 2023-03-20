---
title: react hook form 사용해보기
date: 2023-03-19 19:00:01 +0900
categories: [Dev]
tags: [react, typescript, hook, react hook]

---

# React Hook Form을 사용해보기

아래와 같이 간단한 투두리스트 input폼을 useState를 이용하여 만들어 보았다.  
만약 앱의 규모가 커진다면 너무 많은 state들을 작성해줘야 하고, 많은 form validation(검증) 이 필요해진다.

``` react
import { useState } from "react";

function ToDoList() {
    const [toDo, setToDo] = useState("");
    const [toDoError, setToDoError] = useState("");
    const onChange = (event: React.FormEvent<HTMLInputElements>) => {
        const {currentTarget : {value}} = event;
        setToDo(value);
    };
    
    const onSubmit = (event: React.FormEvent<HTMLFormElement>) => {
        event.preventDefault();
        if (toDo.length < 10) {
            return setToDoError("ToDo must be longer");
        }
        console.log("submit");
    }
    
    return (
        <div>
        	<form onSubmit={onSubmit}>
        		<input value={toDo} onChange={onChange} type="text" placeholder="write ToDo" />
                    <button>Add</button>
					{toDoError !== "" ? toDoError : null}
        	</form>
        </div>
    );
}

export default ToDoList;
```

같은 것을 react hook form을 이용하여 간추릴 수 있다.

```bash
npm install react-hook-form
```

``` react
import { useState } from "react";
import { useForm } from "react-hook-form";

function ToDoList() {
    const { register, watch} = useForm();
    console.log(watch());
    
    return (
        <div>
        	<form>
        		<input {...register("toDo")} placeholder="write ToDo" />
                    <button>Add</button>
        	</form>
        </div>
    );
}

export default ToDoList;
```



**watch함수로 input에 입력한 값들의 변화를 관찰할수 있다.**  
console.log(watch()); 해보면  
key로 toDo를, value로 input의 입력값을 가진 객체를 확인할 수 있다.  

**register 함수로 onChange이벤트와 value, useState를 모두 대체할 수 있다.**  
form이 커지면 커질 수록 더욱 효과적이다.  

``` react
const { register, watch} = useForm();
```

이 한 줄로 새로운 input을 만들 때마다 register함수만 실행해주면,  
register함수가 그 input을 어떠한 객체에 값으로 준다. 그리고 그 객체를 input의 props로 사용할 수 있다.  

<hr>

# React Hook Form으로 유효성 검사하기

``` react
const { register, handleSubmit } = useForm();
```

**handleSubmit 함수가 유효성 검사를 담당한다.**

사용법은 form태그의 onSubmit이벤트 안에 handleSubmit을 호출하고, 2개의 인자를 받는다.  
 ```react
 handleSubmit: ((data: Object, event?: Event) => void, (errors: Object, event?: Event) => void) => Function
 //이 함수는 form 유효성 검사가 성공하면 form 데이터를 받습니다.
 ```

하나는 데이터가 유효할 때 호출되는 함수(onValid), 다른 하나는 유효하지 않을 때(onInvalid). 후자는 선택사항.

```react
const { register, handleSubmit } = useForm<IFormData>();
const onValid = (data: IFormData) => console.log(data);
<form onSubmit={handleSubmit(onValid)}>
```

유저가 submit하면 handleSubmit은 해야하는 모든 유효성 검사를 끝낸 이후에 우리의 데이터가 유효할 때만 우리의 함수를 호출한다.  
handleSubmit으로 유효성 검사를 하면 에러가 있는 input이 있는 경우 해당 input칸에 즉시 마우스 커서를 이동(focus)시켜주기도 한다.  

```react
<input {...register("password", { required: true, minLength: 10})} />
```

이전 방식처럼 onSubmit이벤트 함수를 직접 만들 때는 submit이벤트에서 유효성 조건을 적어줬다.  
하지만 react-hook-form을 사용할 때는 위의 코드처럼만 작성해주면  
react-hook-form이  알아서 이전 방식처럼 조건을 확인하고 유효성 검사를 진행한다.



**formState함수**

```react
const { register, formState } = useForm<IFormData>();
console.log(formState.errors);
```

에러가 발생할 시 에러가 어떤 에러인지 콘솔을 통해 확인할 수 있다.

```react
<input {...register("email", { required: "email required", 
                                 minLength: {
                                     value: 8,
                                     message: "email is too short"
                                 },
                                 pattern: /^[A-Za-z0-9._%+-]+@naver.com )} />
```

required: true 라고 쓰는 것 대신에 "" 안에 에러 메시지를 보낼 수도 있다. 이때 기본으로 required: true 이다.  
minLength의 경우엔 객체로 보낼 수도 있다.  
pattern에 정규식을 사용해 형식 조건을 줄 수 있다.

<hr>

# 유효성 검사 사용자화

```react
setError:(name: string, error: FieldError, { shouldFocus?: boolean }) => void;
```

**setError 함수를 사용해 하나 이상의 error를 수동으로 설정할 수 있다.**

```react
const onValid = (data: IFormData) => {
    if (data.password !== data.password1) {
        return setError(
        "password1",
        { message: "비밀번호가 일치하지 않습니다." },
        { shouldFocus: true }
        );
    }
}
console.log(formState.errors);
```


**validate로 원하는 어떠한 규칙이든 (내가 만든 함수로) 검사할 수 있다.**

username value에 "GM" 이라는 글자가 절대 포함되지 않게 하고 싶을 때 다음과 같이 작성할 수 있다.

```react
<input {...register("username", {
        required: "username required",
        validate: {
            noGm : (value) => value.includes("GM") ? "GM은 포함할 수 없음." : true,
        },
    })} placeholder="Username"/>
```

<hr>



1. useForm을 import한다.
   ```react
   import { useForm } from "react-hook-form";
   ```

2. 컴포넌트에서 useForm을 호출하면 register와 handleSubmit함수가 제공된다.

3. register함수를 form에 있는 모든 input에서 호출한다.

4. 원한다면 검사 옵션을 설정할 수 있다. required 등등

5. 