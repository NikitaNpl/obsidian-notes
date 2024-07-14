# SOLID в REACT 2024-07-09

**_Теги_:** #ПринципыРазработки #Разработка #ПО #SOLID #Frontend #React
***Ссылки*:** [[SOLID]]
## Single Responsibility Principle (S) — принцип единой ответственности

Для каждого класса/функции/компонента должно быть определено единственное назначение.

> [!FAQ]- Как определить выполняет ли наша функция данный принцип?
> Функция имеет единственное назначение, если вы не можете осмысленно извлечь из нее другую функцию. Соответсвенно, если вы можете извлечь другую функцию, то исходная функция делала больше, чем одно действие. (Роберт Мартин, "Чистый код")
### Цели принципа единой ответственности  

- разбивать большие компоненты на более мелкие;
- извлекать код, не связанный с функциональностью основного компонента, в отдельные служебные функции;
- выность бизнес-логику компонентов в пользовательские хуки.
- получение читаемости кода;
- наличие простоты тестирования;
- возможность проще отлавливать баги;
- наличие простоты доработки.
### Пример принципа единой ответственности  

В контексте React-проектов, обычно может намекать на то, что что-то происходит не так с точки зрения данного принципа, будет размера компонентов.

Рассмотрим компонент `TodosPage`, задачей такого компонента является в формировании страницы. В примере получаются данные по `todo` элементам, затем они отправляются в разметку.

```jsx
const TodosPage = () => {
	const [todos, setTodos] = useState([]);

	useEffect(() => {
		async function getTodos() {
			const { data } = await axios.get("https://someapi/todos");
			setTodos(data);
		};
		getTodos();
	}, []);

	return (
		<div>
			<h1>My todos:</h1>
			<ul>
				{todos.map(todo => (
					<li key={todo.id}>{todo.title}</li>
				))}
			</ul>
		</div>
	);
};
```

Чтобы улучшить данный компонент, можно из него вынести логику получения `todo` элементов в отдельный хук или контейнер, и логику с разметкой в виде обособленного компонента. 

```jsx
async function getTodos() {
	const { data } = await axios.get("https://someapi/todos");
	return data;
};

function useTodos() {
	const [todos, setTodos] = useState([]);

	useEffect(() => {
		getTodos().then(data => setTodos(data));
	}, []);

	return todos;
};

const TodosPage = () => {
	const todos = useTodos();

	return (
		<div>
			<h1>My todos:</h1>
			<TodoList todos={todos} />
		</div>
	);
};

const TodosList = ({ todos }) => (
	<ul>
		{todos.map(todo => <TodoItem key={todo.id} todo={todo}/>)}
	</ul>
);

const TodoItem = ({ title }) => (
	<li>{title}</ul>
);
```

## Open-Closed Principle (O) — принцип открытости-закрытости

Программные сущности должны быть открыты для расширения, но закрыты для модификации. В контексте React-а — это использование композиции компонентов.
### Цели принципа открытости-закрытости

- упрощение добавления новых функций в систему, не влияя на существующую функциональность;
- создание поддерживаемой и расширяемой системы;
- уменьшение риска ошибок в уже созданной функциональности.
### Пример принципа открытости-закрытости

К примеру, у нас есть реализация простого `Input` с возможностью простого ввода текста. Затем, к нам приходит заказчик и говорит, что желает видеть префиксы для определенных форматов вводимых данных (номера телефона или url). К нам может прийти еще иное требование, по добавлению иконок справа или слева от поля ввода, также могут быть различные виды кнопок. Перед нами уже встает вопрос, мы либо дорабатываем текущий компонент, либо идем по какому-либо другому пути. На начальных этапах, мы можем добавить дополнительные свойства к компоненту и все будет прекрасно работать. Но чем дальше мы заходим, чем больше требований приходят, то изначальный простой компонент становится очень уж сложным.

Варианты расширения компонентов:

- дополнительные свойства;
- свойство `children`;
- подход `renderProps` (когда свойство принимает на вход некоторую функцию и возвращает разметку);
- передача в свойства готовые `JSX` разметку;
- паттерн compound components (составные компоненты).

Хорошим примером использования паттерна compound components является библиотека [chackra ui](https://v2.chakra-ui.com/docs/components/input/usage)

Рассмотрим элементы ввода, которые должны следующим образом
![[Pasted image 20240711191241.png]]

Это просто `Input` который может по-разному отображаться, иметь разные размеры и границ, значения по умолчанию и различные обработчики событий. Но для такого компонента количество свойств может быть в районе десятка, их не особо много.

```jsx
<Input varian='outline' placeholder='Outline' size='md' />
<Input varian='filled' placeholder='Filled' size='lg' />
<Input varian='flushe' placeholder='Flushed' />
```

В случае, когда у нас есть необходимость добавить элемента справа или слева от элемента ввода, мы можем пойти по пути создания компонента обертки с управлением размеров.
![[Pasted image 20240711191334.png]]

```jsx
<InputGroup>
    <InputLeftAddon>+234</InputLeftAddon>
    <Input type='tel' placeholder='phone number' />
</InputGroup>

<InputGroup size='sm'>
    <InputLeftAddon>https://</InputLeftAddon>
    <Input placeholder='mysite' />
    <InputRightAddon>.com</InputRightAddon>
</InputGroup>
```

С добавлением иконок дела обстоят похожим образом
![[Pasted image 20240711191554.png]]

```jsx
<InputGroup>
    <InputLeftElement pointerEvents='none'>
      <PhoneIcon color='gray.300' />
    </InputLeftElement>
    <Input type='tel' placeholder='Phone number' />
</InputGroup>

<InputGroup>
    <InputLeftElement pointerEvents='none' color='gray.300' fontSize='1.2em'>
      $
    </InputLeftElement>
    <Input placeholder='Enter amount' />
    <InputRightElement>
      <CheckIcon color='green.500' />
	</InputRightElement>
</InputGroup>
```

С добавлением кнопки в рамки элемента ввода
![[Pasted image 20240711192202.png]]

```jsx
const PasswordInput = () => {
  const [show, setShow] = useState(false);
  const handleClick = () => setShow(!show)

  return (
    <InputGroup size='md'>
      <Input type={show ? 'text' : 'password'} placeholder='Enter password' />
      <InputRightElement width='4.5rem'>
        <Button size='sm' onClick={handleClick}>
          {show ? 'Hide' : 'Show'}
        </Button>
      </InputRightElement>
    </InputGroup>
  )
};
```

## Liskov Substitution Principle (L) — принцип подстановки Барбары Лисков

Функции, которые используют базовый тип, должны иметь возможность использовать подтипы базового типа не зная об этом.
### Цели принципа подстановки Барбары Лисков

- в React наследование не используется для уменьшения дублирования кода между компонентами, поэтому вместо этого рекомендуется использовать композицию;
- базовый компонент имеет определенный интерфейс и ожидаемый функционал;
- подкомпонент должен минимально иметь одинаковый интерфейс с функциональностью;
- следуя этому принципу, можно комбинировать наборы простых компонентов, создавая более сложные.
### Пример принципа подстановки Барбары Лисков

```jsx
export interface ButtonProps extends 
	React.ButtonHTMLAttributes<HTMLButtonElement> {}

// Base component
const Button = ({children, ...rest}: ButtonProps): JSX.Element => (
	<button {...rest}>{children}</button>
);

export interface RedButton extends ButtonProps {
	isRed?: boolean;
}

// Derived component
const RedButton = ({
	isRed = false,
	children,
	...rest
}: RedButton): JSX.Element => (
	<Button 
		{...props} 
		style={isRed ? { background: "red" } : undefined}
	>
		{children}
	</Button>
);

const LSP = (): JSX.Element => (
    <div className='button-wrapper'>
        <Button>Click me</Button>
        <RedButton isRed>Click me</RedButton>
    </div>
);
```

## Interface Segregation Principle (I) — принцип разделения интерфейса

Клиенты не должны зависеть от интерфейсов, которые они не используют. 

Для React-приложений это означает, что компоненты не должны зависеть от свойств, которые они не используют. Принцип выступает за минимизацию зависимостей между компонентами системы, делая их менее связанными и, следовательно, более пригодными для повторного использования.
### Цели принципа разделения интерфейсов

- улучшенная сопровождаемость: маленькие и сфокусированные компоненты на конкретных вещах может привести к пониманию кодовой базы. Это позволит сократить время и усилия на изменения в приложении;
- повышенная гибкость: минимизация зависимостей между компонентами облегчает повторное использование и рефакторинг компонентов.
### Пример принципа разделения интерфейса

Для примера, можем рассмотреть компонент, отвечающий за отображения приветственного текста для некоторого пользователя. 

```jsx
interface User {
	name: string;
	age: number;
	role: string;
}

const DisplayGreeting = (props: User): JSX.Element => (
    <div>
        <h1>`Hello, ${props.user.name}!`</h1>
    </div>
);

<DisplayGreeting user={user} />
```

В данном компоненте `DisplayGreeting` есть проблема зависимости от типа `User`, поскольку в свойствах имеется вложенный объект, хотя в самом компоненте используется лишь передаваемое имя пользователя.

К примеру, у нас возникнет желание изменить структуру данных типа `User`, может быть такое, что мы откажемся от плоской структуры данных и добавим какую-нибудь вложенность. Из-за этих изменений, нам придется изменить логику компонента `DisplayGreeting`.

```jsx
interface NewUser {
	personalInfo: {
		name: string;
		age: number;
	}
	role: string;
}

const DisplayGreeting = (props: NewUser): JSX.Element => (
    <div>
        <h1>`Hello, ${props.user.personalInfo.name}!`</h1>
    </div>
);

<DisplayGreeting user={user} />
```

Для того, чтобы избавиться от постоянных изменений компонента, куда лучше переделать сигнатуру нашего компонента `DisplayGreeting`. Цель данного компонента, отобразить строковое значение, которое знает пользователь на момент монтирования нашего компонента.

```jsx
const DisplayGreeting = (
	{ name }: { name: string }
): JSX.Element => (
    <div>
        <h1>`Hello, ${name}!`</h1>
    </div>
);

<DisplayGreeting name={user.personalInfo.name} />
```

## Dependency Inversion Principle (D) — принцип инверсии зависимостей

Принцип говорит о том, что программисту нужно зависеть от абстракций, а не от реализации.

Другими словами, один компонент не должен зависеть от другого, скорее они оба должна зависеть от общей абстракции
### Цели принципа инверсии зависимостей

- модули высокого уровня не должны зависеть от модулей низкого уровня, и те, и другие должны зависеть от абстракций;
- упрощение тестирование за счет изоляций и абстракций от внешних API;
- отделение компонентов друг от друга для дальнейших упрощенных изменений.
### Пример принципа инверсии зависимостей

Рассмотрим пример в логин формой. Перед нами встают вопросы, где у нас будет происходить логика отправки, уведомления об ошибках и какая логика будет реализована при успешном входе пользователя.

Решение в лоб, это реализовать один компонент `LoginForm` с базовой разметкой и логикой обработки запроса на наш API.

```jsx
import axios from 'axios';

const LoginForm = (): JSX.Element => {
	const handleSubmit = (event) => {
		event.preventDefault();
		const data = new FormData(event.currentTarget);
		axios.post('ourUrl', data);	
	};

    return (
	    <form onSubmit={handleSubmit}>
	        <input type="email" />
	        <input type="password" />
	        <button>Войти</button>
	    </div>
    );
};
```

В данном компоненте имеется жесткая привязка к логике обработки наших данных, и в случае, если нам захочется отказаться от библиотеки `axios`, нам придется изменять наш компонент.

Попробуем отказаться от найденной зависимости. Сделаем так, чтобы наш компонент не волновал процесс обработки данных, мы лишь должны предоставлять информацию клиенту, использующий компонент.

```jsx
const LoginFormProps {
	onSubmit: (data: FormData) => void;
}

const LoginForm = ({ onSubmit }: LoginFormProps): JSX.Element => {
	const handleSubmit = (event) => {
		event.preventDefault();
		const data = new FormData(event.currentTarget);
		onSubmit(data);
	};

    return (
	    <form onSubmit={handleSubmit}>
	        <input type="email" />
	        <input type="password" />
	        <button>Войти</button>
	    </div>
    );
};
```

Теперь перед нами встает вопрос, где именно будет реализована логика с отправкой данных на наш API. Есть несколько вариантов, один из которых создание отдельного сервиса и компонента, который собирает воедино логические элементы структуры.

```jsx
// services/sign-in.ts
import axios from 'axios';

export const signIn = (data: FormData) => axios.post('ourUrl', data);

// widgets/Login.tsx
import { signIn } from 'services/sign-in';
import LoginForm from "components/LoginForm";

const Login = (): JSX.Element => (
	    <div>
	        <LoginForm onSubmit={signIN} />
	        <a href="/restore-me">Восстановить пароль</button>
	    </div>
);
```

Другой подход — создания отдельного компонента, который будет выступать в роли абстракции, его основной задачей будет являться подключение компонента нижнего уровня.

```jsx
import axios from 'axios';
import LoginForm from "components/LoginForm";

const ConnectedLoginForm = (): JSX.Element => {
	const onSubmit = (data: FormData) => {
		axios
		.post("ourUrl", data)
		.then(() => console.log("Success"))
		.catch(() => console.log("Error"));
	}

	return (
	    <LoginForm onSubmit={onSubmit} />
	);
};
```