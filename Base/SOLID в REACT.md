# SOLID в REACT 2024-07-09

**_Теги_:** #ПринципыРазработки #Разработка #ПО #SOLID
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
	<Button {...props} style={isRed ? { background: "red" } : undefined}>
		{children}
	</Button>
);

const LSP = (): JSX.Element => (
	<>
        <div className='button-wrapper'>
          <Button>Click me</Button>
          <RedButton isRed>Click me</RedButton>
        </div>
    </>
)
```

