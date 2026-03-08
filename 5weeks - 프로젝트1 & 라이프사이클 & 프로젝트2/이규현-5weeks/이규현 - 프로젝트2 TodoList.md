# TodoList 만들어보기

두번째 프로젝트로는 TodoList를 만들어보자.

## UI 구현

먼저 기능 없이 화면 구조만 잡아보자.

### components/Header.jsx/css

```jsx
import "./Header.css";

const Header = () => {
  return (
    <div className="Header">
      <h3>오늘은 🗓️</h3>
      <h1>{new Date().toDateString()}</h1>
    </div>
  );
};
export default Header;
```

```css
.Header > h1 {
  color: rgb(37, 137, 255);
}
```

현재 날짜를 나타내는 헤더 컴포넌트다. `new Date().toDateString()`으로 오늘 날짜를 렌더링한다.

### components/Editor.jsx/css

```jsx
import "./Editor.css";

const Editor = () => {
  return (
    <div className="Editor">
      <input placeholder="새로운 Todo..." />
      <button>추가</button>
    </div>
  );
};
export default Editor;
```

```css
.Editor {
  display: flex;
  gap: 10px;
}

.Editor input {
  flex: 1;
  padding: 15px;
  border: 1px solid rgb(220, 220, 220);
  border-radius: 5px;
}

.Editor button {
  cursor: pointer;
  width: 80px;
  border: none;
  background-color: rgb(37, 147, 255);
  color: white;
  border-radius: 5px;
}
```

새로운 Todo를 입력하고 추가하는 컴포넌트다. 입력창과 추가 버튼이 `flex`로 배치된다.

### components/List.jsx/css

```jsx
import "./List.css";
import TodoItem from "./TodoItem";

const List = () => {
  return (
    <div className="List">
      <h4>Todo List 📋</h4>
      <input placeholder="검색어를 입력하세요" />
      <div className="todos_wrapper">
        <TodoItem />
        <TodoItem />
        <TodoItem />
      </div>
    </div>
  );
};
export default List;
```

```css
.List {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.List > input {
  width: 100%;
  border: none;
  border-bottom: 1px solid rgb(220, 220, 220);
  padding: 15px 0px;
}

.List > input:focus {
  outline: none;
  border-bottom: 1px solid rgb(37, 147, 255);
}

.List .todos_wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px;
}
```

Todo 목록을 나타내고 검색 기능을 제공하는 컴포넌트다. 여러 `TodoItem`을 렌더링한다.

### components/TodoItem.jsx/css

```jsx
import "./TodoItem.css";

const TodoItem = () => {
  return (
    <div className="TodoItem">
      <input type="checkbox" />
      <div className="content">Todo...</div>
      <div classame="date">Date</div>
      <button>삭제</button>
    </div>
  );
};

export default TodoItem;
```

```css
.TodoItem {
  display: flex;
  align-items: center;
  gap: 20px;
  padding-bottom: 20px;
  border-bottom: 1px solid rgb(240, 240, 240);
}

.TodoItem input {
  width: 20px;
}

.TodoItem .content {
  flex: 1;
}

.TodoItem .date {
  font-size: 14px;
  color: gray;
}

.TodoItem button {
  cursor: pointer;
  color: white;
  background-color: red;
  font-size: 14px;
  border: none;
  border-radius: 5px;
  padding: 5px;
}
```

각각의 Todo 항목을 나타낸다. 체크박스, 내용, 날짜, 삭제 버튼으로 구성돼 있다.

### App.jsx/css

```jsx
import "./App.css";
import Header from "./components/Header";
import Editor from "./components/Editor";
import List from "./components/List";

function App() {
  return (
    <div className="App">
      <Header />
      <Editor />
      <List />
    </div>
  );
}

export default App;
```

```css
body {
  padding: 20px;
}

.App {
  width: 500px;
  margin: 0 auto;

  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 0 auto;
  width: 400px;
}
```

모든 컴포넌트를 조합하는 루트 컴포넌트다.

### UI 화면

![](https://velog.velcdn.com/images/leekh010502/post/3181625f-e18e-4d0b-925d-f5294bd7a131/image.png)

## 기능 구현하기

이제 기능을 추가해보자. 먼저 상태 관리를 위해 `useState`와 `useRef`를 사용한다.

### create- Todo 추가하기

#### App.jsx

```jsx
import "./App.css";
import { useState, useRef } from "react";
import Header from "./components/Header";
import Editor from "./components/Editor";
import List from "./components/List";

const mockDate = [
  {
    id: 0,
    isDone: false,
    content: "React 공부하기",
    date: new Date().getTime(0),
  },
  {
    id: 1,
    isDone: false,
    content: "Node.js 공부하기",
    date: new Date().getTime(0),
  },
  {
    id: 2,
    isDone: false,
    content: "사이드 프로젝트 기획안 작성하기",
    date: new Date().getTime(),
  },
];

function App() {
  const [todos, setTodos] = useState(mockDate);
  const idRef = useRef(3);
  const onCreate = (content) => {
    const newTodo = {
      id: idRef.current++,
      isDone: false,
      content: content,
      date: new Date().getTime(),
    };
    setTodos([newTodo, ...todos]);
  };
  return (
    <div className="App">
      <Header />
      <Editor onCreate={onCreate} />
      <List />
    </div>
  );
}

export default App;
```

- **todos**: Todo 목록을 상태로 관리한다.
- **idRef**: 새 Todo의 id를 관리하기 위해 `useRef`를 사용한다. (렌더링을 유발하지 않음)
- **onCreate**: 새 Todo를 생성해서 배열의 앞에 추가한다.

#### components/Editor.jsx

```jsx
import "./Editor.css";
import { useState, useRef } from "react";

const Editor = ({ onCreate }) => {
  const [content, setContent] = useState("");
  const contentRef = useRef();

  const onChangeContent = (e) => {
    setContent(e.target.value);
  };
  const onKeydown = (e) => {
    if (e.keyCode === 13) {
      onSubmit();
    }
  };
  const onSubmit = () => {
    if (content === "") {
      contentRef.current.focus();
      return;
    }
    onCreate(content);
    setContent("");
  };
  return (
    <div className="Editor">
      <input
        ref={contentRef}
        value={content}
        onKeyDown={onKeydown}
        onChange={onChangeContent}
        placeholder="새로운 Todo..."
      />
      <button onClick={onSubmit}>추가</button>
    </div>
  );
};
export default Editor;
```

- **content**: 입력값을 상태로 관리한다.
- **contentRef**: 입력창에 포커스를 주기 위해 사용한다.
- **onKeydown**: Enter 키(keyCode 13)를 감지해서 Todo를 추가한다.
- **onSubmit**: 내용이 비어있지 않으면 `onCreate` 함수를 호출하고, 입력값을 초기화한다.

#### 결과 화면

![](https://velog.velcdn.com/images/leekh010502/post/82520e14-2491-44ce-840a-effca2f86f5c/image.png)

### TodoList 렌더링하기 + 검색하기

#### App.jsx

```jsx
function App() {
  const [todos, setTodos] = useState(mockDate);
  const idRef = useRef(3);

  const onCreate = (content) => {
    const newTodo = {
      id: idRef.current++,
      isDone: false,
      content: content,
      date: new Date().getTime(),
    };
    setTodos([newTodo, ...todos]);
  };

  return (
    <div className="App">
      <Header />
      <Editor onCreate={onCreate} />
      <List todos={todos} />
    </div>
  );
}
```

`todos` 상태를 `List` 컴포넌트에 props로 전달한다.

#### List.jsx

```jsx
import "./List.css";
import TodoItem from "./TodoItem";
import { useState } from "react";

const List = ({ todos }) => {
  const [search, setSearch] = useState("");

  const onChangeSearch = (e) => {
    setSearch(e.target.value);
  };

  const getFilteredDate = () => {
    if (search === "") {
      return todos;
    }
    return todos.filter((todo) =>
      todo.content.toLowerCase().includes(search.toLowerCase()),
    );
  };
  const filteredTodos = getFilteredDate();

  return (
    <div className="List">
      <h4>Todo List 📋</h4>
      <input
        value={search}
        onChange={onChangeSearch}
        placeholder="검색어를 입력하세요"
      />
      <div className="todos_wrapper">
        {filteredTodos.map((todo) => {
          return <TodoItem key={todo.id} {...todo} />;
        })}
      </div>
    </div>
  );
};
export default List;
```

- **search**: 검색 입력값을 상태로 관리한다.
- **getFilteredDate()**: 검색어에 맞는 Todo만 필터링한다. (대소문자 구분 없음)
- **filteredTodos.map()**: 필터링된 Todo 목록을 렌더링한다.
- **{...todo}**: Todo 객체의 모든 속성을 props로 전달한다.

#### TodoItem.jsx

```jsx
import "./TodoItem.css";

const TodoItem = ({ id, isDone, content, date }) => {
  return (
    <div className="TodoItem">
      <input readOnly checked={isDone} type="checkbox" />
      <div className="content">{content}</div>
      <div classame="date">{new Date(date).toLocaleDateString()}</div>
      <button>삭제</button>
    </div>
  );
};

export default TodoItem;
```

- **destructuring**: props에서 필요한 값들을 바로 추출한다.
- **new Date(date).toLocaleDateString()**: 타임스탬프를 로컬 날짜 형식으로 변환한다.

### TodoList 수정하기

#### App.jsx

```jsx
function App() {
  const [todos, setTodos] = useState(mockDate);
  const idRef = useRef(3);

  const onCreate = (content) => {
    const newTodo = {
      id: idRef.current++,
      isDone: false,
      content: content,
      date: new Date().getTime(),
    };
    setTodos([newTodo, ...todos]);
  };

  const onUpdate = (targetId) => {
    setTodos(
      todos.map((todo) =>
        todo.id === targetId ? { ...todo, isDone: !todo.isDone } : todo,
      ),
    );
  };

  return (
    <div className="App">
      <Header />
      <Editor onCreate={onCreate} />
      <List todos={todos} onUpdate={onUpdate} />
    </div>
  );
}
```

- **onUpdate**: 특정 Todo의 isDone 상태를 토글한다.
  **.map()**: 모든 Todo를 순회하면서 해당 id의 Todo만 isDone 값을 반전시킨다.

#### components/List.jsx

```jsx
const List = ({ todos, onUpdate, onDelete }) => {
  // ... 이전 코드 생략

  return (
    <div className="List">
      <h4>Todo List 📋</h4>
      <input
        value={search}
        onChange={onChangeSearch}
        placeholder="검색어를 입력하세요"
      />
      <div className="todos_wrapper">
        {filteredTodos.map((todo) => {
          return (
            <TodoItem
              key={todo.id}
              {...todo}
              onUpdate={onUpdate}
              onDelete={onDelete}
            />
          );
        })}
      </div>
    </div>
  );
};
```

`onUpdate` 함수를 props로 받아서 `TodoItem`에 전달한다.

#### components/TodoItem.jsx

```jsx
import "./TodoItem.css";

const TodoItem = ({ id, isDone, content, date, onUpdate }) => {
  const onChangeCheckbox = () => {
    onUpdate(id);
  };

  return (
    <div className="TodoItem">
      <input onChange={onChangeCheckbox} checked={isDone} type="checkbox" />
      <div className="content">{content}</div>
      <div className="date">{new Date(date).toLocaleDateString()}</div>
      <button>삭제</button>
    </div>
  );
};

export default TodoItem;
```

- **onChangeCheckbox**: 체크박스 변경 시 `onUpdate` 함수를 호출한다.
- **checked={isDone}**: 체크박스 상태를 `isDone` 값과 동기화한다.

#### 결과 화면

![](https://velog.velcdn.com/images/leekh010502/post/e81f769e-460d-4934-808f-a716993cde98/image.png)

![](https://velog.velcdn.com/images/leekh010502/post/52c0188b-acc8-4dd5-8f27-76d471b489e3/image.png)

### TodoList 삭제하기

#### App.jsx

```jsx
function App() {
  const [todos, setTodos] = useState(mockDate);
  const idRef = useRef(3);

  const onCreate = (content) => {
    const newTodo = {
      id: idRef.current++,
      isDone: false,
      content: content,
      date: new Date().getTime(),
    };
    setTodos([newTodo, ...todos]);
  };

  const onUpdate = (targetId) => {
    setTodos(
      todos.map((todo) =>
        todo.id === targetId ? { ...todo, isDone: !todo.isDone } : todo,
      ),
    );
  };

  const onDelete = (targetId) => {
    setTodos(todos.filter((todo) => todo.id !== targetId));
  };

  return (
    <div className="App">
      <Header />
      <Editor onCreate={onCreate} />
      <List todos={todos} onUpdate={onUpdate} onDelete={onDelete} />
    </div>
  );
}
```

- **onDelete**: `filter`를 사용해서 해당 id를 가진 Todo를 제외한 새로운 배열을 반환한다.

#### components/List.jsx

```jsx
import "./List.css";
import TodoItem from "./TodoItem";
import { useState } from "react";

const List = ({ todos, onUpdate, onDelete }) => {
  const [search, setSearch] = useState("");

  const onChangeSearch = (e) => {
    setSearch(e.target.value);
  };

  const getFilteredDate = () => {
    if (search === "") {
      return todos;
    }
    return todos.filter((todo) =>
      todo.content.toLowerCase().includes(search.toLowerCase()),
    );
  };
  const filteredTodos = getFilteredDate();

  return (
    <div className="List">
      <h4>Todo List 📋</h4>
      <input
        value={search}
        onChange={onChangeSearch}
        placeholder="검색어를 입력하세요"
      />
      <div className="todos_wrapper">
        {filteredTodos.map((todo) => {
          return (
            <TodoItem
              key={todo.id}
              {...todo}
              onUpdate={onUpdate}
              onDelete={onDelete}
            />
          );
        })}
      </div>
    </div>
  );
};
export default List;
```

`onDelete` 함수를 props로 받아서 `TodoItem`에 전달한다.

#### components/TodoItem.jsx

```jsx
import "./TodoItem.css";

const TodoItem = ({ id, isDone, content, date, onUpdate, onDelete }) => {
  const onChangeCheckbox = () => {
    onUpdate(id);
  };

  const onClickDeleteButton = () => {
    onDelete(id);
  };

  return (
    <div className="TodoItem">
      <input onChange={onChangeCheckbox} checked={isDone} type="checkbox" />
      <div className="content">{content}</div>
      <div className="date">{new Date(date).toLocaleDateString()}</div>
      <button onClick={onClickDeleteButton}>삭제</button>
    </div>
  );
};

export default TodoItem;
```

- **onClickDeleteButton**: 삭제 버튼 클릭 시 `onDelete` 함수를 호출한다.

#### 결과화면

![](https://velog.velcdn.com/images/leekh010502/post/e1e1667a-78a5-41a7-8aed-b94d2b5bcc27/image.png)
