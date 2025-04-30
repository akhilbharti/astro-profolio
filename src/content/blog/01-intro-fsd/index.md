---
title: "Mastering Feature Slice Methodology in React TypeScript"
description: "A practical guide to implementing Feature Slice Methodology in React TypeScript, using a To-Do List application as an example."
slug: "feature-slice-methodology-react-typescript"
date: "Sep 17 2023"
---


## Introduction

In the fast-paced world of web development, staying organized and maintaining codebase scalability is essential. Feature Slice Methodology is a modern approach that can help you achieve just that. In this blog post, we will delve into Feature Slice Methodology, provide a real-world example, and demonstrate its implementation using React TypeScript.

## Understanding Feature Slice Methodology

Feature Slice Methodology is a software architectural pattern that encourages the structuring of your codebase into self-contained feature slices. Each feature slice encapsulates all the necessary components, styles, logic, and state related to a specific feature or functionality of your application. This methodology has gained popularity due to its ability to enhance code maintainability, scalability, and collaboration among development teams.


![FSD Image](./Mastering%20Feature%20Slice%20Methodology%20in%20React%20TypeScript.webp)

### Key Benefits of Feature Slice Methodology:
- **Modularity**: Features are isolated, making it easier to develop, test, and maintain them individually.
- **Scalability**: Scalability is simplified as features can be added or removed with minimal impact on the rest of the application.
- **Collaboration**: Teams can work concurrently on different features without stepping on each other's toes.
- **Readability**: The codebase becomes more readable and easier to navigate.

## Example: To-Do List Application

Let's illustrate Feature Slice Methodology with a simple example: a To-Do List application. We will structure the application using this methodology.

### Project Setup

Before we start, make sure you have Node.js and npm (or yarn) installed on your system. Then, follow these steps:

1. **Create a new React TypeScript project** using Create React App:
    ```bash
    npx create-react-app todo-list --template typescript
    ```

2. **Change to the project directory**:
    ```bash
    cd todo-list
    ```

3. **Install the necessary dependencies**:
    ```bash
    npm install @reduxjs/toolkit react-redux
    ```

### Structuring the Application

Our To-Do List application will consist of two main feature slices: "Tasks" and "Filters."
 ```md
 my-app/
│
├── src/
│   ├── app/
│   │   ├── store.ts
│   │   └── App.tsx
│   │
│   ├── features/
│   │   ├── filters/
│   │   │   ├── FilterBar.tsx
│   │   │   └── filterSlice.ts
│   │   │
│   │   ├── tasks/
│   │   │   ├── TaskItem.tsx
│   │   │   ├── TaskList.tsx
|   |   |   └── tasksSlice.ts
│   │   │
│   │   └── ...
│   │
│   ├── common/
│   │   ├── CommonComponent1.tsx
│   │   ├── CommonComponent2.tsx
│   │   └── ...
│   │
│   └── index.tsx
│
├── public/
│   ├── index.html
│   ├── manifest.json
│   └── ...
│
├── package.json
├── tsconfig.json
├── README.md
└── ...
```


### 1. Tasks Feature Slice

Create a directory for the "Tasks" feature slice:

```bash
mkdir src/features/tasks
```
Inside the “tasks” directory, create the following files:
- `tasksSlice.ts`: Define the Redux slice for tasks.

```typescript
// src/features/tasks/tasksSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

export interface Task {
  id: number;
  text: string;
  completed: boolean;
}

interface TasksState {
  tasks: Task[];
}

const initialState: TasksState = {
  tasks: [],
};

const tasksSlice = createSlice({
  name: 'tasks',
  initialState,
  reducers: {
    addTask: (state, action: PayloadAction<string>) => {
      const newTask: Task = {
        id: state.tasks.length + 1,
        text: action.payload,
        completed: false,
      };
      state.tasks.push(newTask);
    },
    toggleTaskCompletion: (state, action: PayloadAction<number>) => {
      const task = state.tasks.find((t) => t.id === action.payload);
      if (task) {
        task.completed = !task.completed;
      }
    },
    deleteTask: (state, action: PayloadAction<number>) => {
      state.tasks = state.tasks.filter((task) => task.id !== action.payload);
    },
  },
});

export const { addTask, toggleTaskCompletion, deleteTask } = tasksSlice.actions;

export default tasksSlice.reducer;
```

- `TaskList.tsx`: Implement the task list component.
```typescript

// src/features/tasks/TaskList.tsx
import React, { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { RootState } from '../../app/store';
import TaskItem from './TaskItem';
import { addTask } from './tasksSlice';

const TaskList: React.FC = () => {
  const showCompleted = useSelector(
    (state: RootState) => state.filters.showCompleted
  );
  const tasks = useSelector((state: RootState) =>
    showCompleted ? state.tasks.tasks : state.tasks.tasks.filter(task => !task.completed)
  );

  const dispatch = useDispatch();
  const [newTaskText, setNewTaskText] = useState('');

  const handleAddTask = () => {
    if (newTaskText.trim() !== '') {
      dispatch(addTask(newTaskText));
      setNewTaskText(''); // Clear the input field
    }
  };

  return (
    <div>
      <h2>Task List</h2>
      <div>
        <input
          type="text"
          placeholder="Add a new task"
          value={newTaskText}
          onChange={(e) => setNewTaskText(e.target.value)}
        />
        <button onClick={handleAddTask}>Add</button>
      </div>
      <ul>
        {tasks.map((task) => (
          <TaskItem key={task.id} task={task} />
        ))}
      </ul>
    </div>
  );
};

export default TaskList;
```
- `TaskItem.tsx`: Implement the task item component.
```typescript
// src/features/tasks/TaskItem.tsx
import React from 'react';
import { useDispatch } from 'react-redux';
import { Task, toggleTaskCompletion, deleteTask } from './tasksSlice';

interface TaskItemProps {
  task: Task;
}

const TaskItem: React.FC<TaskItemProps> = ({ task }) => {
  const dispatch = useDispatch();

  const handleToggleTaskCompletion = () => {
    dispatch(toggleTaskCompletion(task.id));
  };

  const handleDeleteTask = () => {
    dispatch(deleteTask(task.id));
  };

  return (
    <li>
      <input
        type="checkbox"
        checked={task.completed}
        onChange={handleToggleTaskCompletion}
      />
      <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
        {task.text}
      </span>
      <button onClick={handleDeleteTask}>Delete</button>
    </li>
  );
};

export default TaskItem;
```
### 2. Filters Feature Slice
Create a directory for the “Filters” feature slice:

```bash
mkdir src/features/filters
```

Inside the “filters” directory, create the following files:

- `filtersSlice.ts`: Define the Redux slice for filters.

```typescript
// src/features/filters/filtersSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

export interface FiltersState {
  showCompleted: boolean;
}

const initialState: FiltersState = {
  showCompleted: true, // Initially show completed tasks
};

const filtersSlice = createSlice({
  name: 'filters',
  initialState,
  reducers: {
    toggleShowCompleted: (state) => {
      state.showCompleted = !state.showCompleted;
    },
  },
});

export const { toggleShowCompleted } = filtersSlice.actions;

export default filtersSlice.reducer;
```

- `FilterBar.tsx`: Implement the filter bar component.

```typescript
// src/features/filters/FilterBar.tsx
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { RootState } from '../../app/store';
import { toggleShowCompleted } from './filterSlice';

const FilterBar: React.FC = () => {
  const showCompleted = useSelector(
    (state: RootState) => state.filters.showCompleted
  );
  const dispatch = useDispatch();

  const handleToggleShowCompleted = () => {
    dispatch(toggleShowCompleted());
  };

  return (
    <div>
      <label>
        <input
          type="checkbox"
          checked={showCompleted}
          onChange={handleToggleShowCompleted}
        />
        Show Completed Tasks
      </label>
    </div>
  );
};

export default FilterBar;
```

###Redux Store Configuration

To complete the Redux setup, configure the Redux store in your `store.ts` or `app/store.ts` file.

```typescript
// src/app/store.ts
import { configureStore } from '@reduxjs/toolkit';
import tasksReducer from '../features/tasks/tasksSlice';
import filtersReducer from '../features/filters/filtersSlice';

export const store = configureStore({
  reducer: {
    tasks: tasksReducer,
    filters: filtersReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Integrating Components

Finally, integrate the `TaskList` component and other components into your main application component (e.g., `App.tsx`).

```typescript
// src/App.tsx
import React from 'react';
import { Provider } from 'react-redux';
import { store } from './app/store';
import TaskList from './features/tasks/TaskList';
import FilterBar from './features/filters/FilterBar';

function App() {
  return (
    <Provider store={store}>
      <div className="App">
        <h1>ToDo List App</h1>
        <TaskList />
        <FilterBar />
      </div>
    </Provider>
  );
}

export default App;
```
## Conclusion
Feature Slice Methodology is a powerful approach to organizing your React TypeScript projects. By dividing your application into self-contained feature slices, you can enhance code maintainability, scalability, and collaboration within your development team.

In this blog post, we’ve created a simple To-Do List application to demonstrate how to apply Feature Slice Methodology in a real-world scenario. Feel free to expand upon this example in your own projects, and witness the benefits of this methodology firsthand. Happy coding!