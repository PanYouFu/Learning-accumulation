> 如何测试自定义 hooks

要在本地测试自定义 React hook，除了新建一个完整的 React 项目之外，还有一些其他方法。以下是几种常见的方案：

### 1. 使用 React Testing Library

React Testing Library 是一个强大的测试工具，可以帮助你测试 React 组件和自定义 hook。你可以使用它来编写单元测试，从而在不创建完整应用的情况下测试你的 hook。

**步骤**：

1. **创建一个新的项目文件夹**：

   ```sh
   mkdir my-hook-testing
   cd my-hook-testing
   npm init -y
   ```
   
2. **安装依赖**：

   ```sh
   npm install react react-dom @testing-library/react @testing-library/jest-dom jest
   ```
   
3. **设置 Jest 配置**（可选）： 在 `package.json` 中添加 Jest 配置：

   ```json
   "scripts": {
     "test": "jest"
   },
   "jest": {
     "testEnvironment": "jsdom"
   }
   ```
   
4. **编写你的自定义 hook**： 在 `src` 文件夹中创建一个文件，例如 `useMyHook.js`：

   ```js
   import { useState } from 'react';
   
   function useMyHook() {
     const [value, setValue] = useState(0);
   
     const increment = () => setValue(prev => prev + 1);
     const decrement = () => setValue(prev => prev - 1);
   
     return { value, increment, decrement };
   }
   
   export default useMyHook;
   ```
   
5. **编写测试用例**： 在 `src` 文件夹中创建一个测试文件，例如 `useMyHook.test.js`：

   ```javascript
   import { renderHook, act } from '@testing-library/react-hooks';
   import useMyHook from './useMyHook';
   
   test('should increment and decrement value', () => {
     const { result } = renderHook(() => useMyHook());
   
     expect(result.current.value).toBe(0);
   
     act(() => {
       result.current.increment();
     });
     expect(result.current.value).toBe(1);
   
     act(() => {
       result.current.decrement();
     });
     expect(result.current.value).toBe(0);
   });
   ```
   
6. **运行测试**：

   ```sh
   npm test
   ```

### 2. 使用 Storybook

Storybook 是一个独立的工具，可以让你在隔离环境中开发和测试 UI 组件。你可以使用它来测试你的 hook。

**步骤**：

1. **在现有项目中安装 Storybook**：

   ```
   sh
   复制代码
   npx sb init
   ```

2. **创建一个 story 来测试你的 hook**： 在 `src` 文件夹中创建一个文件，例如 `useMyHook.stories.js`：

   ```
   javascript
   复制代码
   import React from 'react';
   import { useMyHook } from './useMyHook';
   
   export default {
     title: 'Example/useMyHook',
   };
   
   export const Default = () => {
     const { value, increment, decrement } = useMyHook();
     return (
       <div>
         <p>Value: {value}</p>
         <button onClick={increment}>Increment</button>
         <button onClick={decrement}>Decrement</button>
       </div>
     );
   };
   ```

3. **运行 Storybook**：

   ```
   sh
   复制代码
   npm run storybook
   ```

### 3. 使用 [CodeSandbox](https://codesandbox.io/)

CodeSandbox 是一个在线代码编辑器，允许你快速创建、共享和测试 React 代码。

1. **创建一个新的 sandbox**：
   - 访问 [CodeSandbox](https://codesandbox.io/) 并选择 `React` 模板。
2. **编写你的自定义 hook 并测试**：
   - 在 `src` 文件夹中创建你的 hook 文件和测试文件。

### 总结

虽然新建一个 React 项目是测试自定义 hook 的一种方法，但使用 React Testing Library、Storybook 或 CodeSandbox 等工具可以更方便、更高效地测试你的 hook，而无需创建一个完整的项目。这些方法不仅节省时间，还可以提高你的开发和测试效率。



---

### 1. 创建新的 React + TypeScript 项目

使用 Create React App 并指定 TypeScript 模板：

```shell
npx create-react-app my-hook-testing --template typescript
cd my-hook-testing
```

### 2. 编写自定义 Hook

在 `src` 文件夹中创建一个 TypeScript 文件来编写你的自定义 Hook。例如，创建 `useMyHook.ts`：

```ts
// src/useMyHook.ts
import { useState } from 'react';

function useMyHook() {
  const [value, setValue] = useState<number>(0);

  const increment = () => setValue(prev => prev + 1);
  const decrement = () => setValue(prev => prev - 1);

  return { value, increment, decrement };
}

export default useMyHook;
```

### 3. 创建测试组件

创建一个组件来使用和测试这个 Hook。例如，创建 `HookTester.tsx`：

```ts
// src/HookTester.tsx
import React from 'react';
import useMyHook from './useMyHook';

const HookTester: React.FC = () => {
  const { value, increment, decrement } = useMyHook();

  return (
    <div>
      <p>Value: {value}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}

export default HookTester;
```

### 4. 在应用程序中使用测试组件

打开 `src/App.tsx`，导入并使用 `HookTester` 组件：

```ts
// src/App.tsx
import React from 'react';
import HookTester from './HookTester';

const App: React.FC = () => {
  return (
    <div className="App">
      <header className="App-header">
        <HookTester />
      </header>
    </div>
  );
}

export default App;
```

### 5. 运行项目

启动开发服务器以查看和测试你的 Hook：

```sh
npm start
```

### 6. 添加单元测试

使用 React Testing Library 和 Jest 来编写单元测试。首先，安装 React Testing Library：

```sh
npm install @testing-library/react @testing-library/jest-dom @testing-library/react-hooks
```

然后，在 `src` 文件夹中创建一个测试文件 `useMyHook.test.ts`：

```
typescript
复制代码
// src/useMyHook.test.ts
import { renderHook, act } from '@testing-library/react-hooks';
import useMyHook from './useMyHook';

test('should increment and decrement value', () => {
  const { result } = renderHook(() => useMyHook());

  expect(result.current.value).toBe(0);

  act(() => {
    result.current.increment();
  });
  expect(result.current.value).toBe(1);

  act(() => {
    result.current.decrement();
  });
  expect(result.current.value).toBe(0);
});
```

### 7. 运行测试

运行测试来验证你的 Hook 是否正常工作：

```sh
npm test
```

### 总结

通过上述步骤，你可以创建一个新的 React + TypeScript 项目，编写和测试你的自定义 Hook。这种方法可以让你在一个独立的环境中进行开发和测试，同时借助 Create React App 和 TypeScript 的便利，快速启动项目并使用内置的开发服务器和测试工具。