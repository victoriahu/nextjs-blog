---
title: "Solving React Application Performance Challenges with useMemo and useCallback"
date: "2023-10-02"
---
# Solving React Application Performance Challenges with useMemo and useCallback

React, the popular JavaScript library for building user interfaces, is known for its simplicity and declarative approach. However, as your applications grow in complexity, you may encounter performance challenges that need to be addressed. Two essential tools in your React toolbox for optimizing performance are useMemo and useCallback. But what is the difference between them? When do we use them? Let's explore the problems these hooks solve and how to use them effectively in your React applications.


## React's Dependency System

Before we delve into useMemo and useCallback, it's important to understand React's dependency tracking system. 

In React, when you pass an array or an object as a dependency to a useEffect, React will use a strict equality (===) comparison to determine whether the dependencies have changed. This means that React will consider the dependency to have changed if a new reference to an array or an object is provided, even if the contents of the array or the properties of the object remain the same.

Let's look at a more detailed explanation of this behavior:

### Array as a Dependency:

If you pass an array as a dependency, like this:

```jsx
useEffect(() => {
  // Your effect logic here
}, [myArray]);
```
React will compare the current myArray reference to the previous reference during each render. If a new array reference is created (even if the elements in the array are the same), React will consider the dependency to have changed, and it will trigger the effect to run. This is because, from React's perspective, the new array reference is not strictly equal (===) to the previous reference.

For example, if you have:

```jsx
const prevArray = [1, 2, 3];
const newArray = [1, 2, 3];
```

Even though newArray has the same elements as prevArray, React will consider them different when passed as dependencies to useEffect.

In React, the effect or function will always rerun because the new array is considered a different object in memory.

### Object as a Dependency:

The same principle applies when passing an object as a dependency:

```jsx
useEffect(() => {
  // Your effect logic here
}, { key: 'value' });
```

React will compare the current object reference to the previous reference. If a new object reference is created (even if the properties and values within the object are the same), React will trigger the effect to run.

For example:

```jsx
const prevObject = { key: 'value' };
const newObject = { key: 'value' };
```

React will consider newObject and prevObject as different references when passed as dependencies to useEffect.

Again, the effect or function will always rerun because the new object is considered a different object in memory.

## Problem: Expensive Computations

In more complex React applications, you might encounter components that perform expensive computations, such as sorting, filtering, or other data processing tasks. If these computations are executed on every render, it can lead to performance bottlenecks, especially with large datasets.

### The Solution: useMemo
useMemo is a React hook that solves the problem of expensive computations. It memoizes the result of an expensive operation and ensures that the operation is executed only when its dependencies change.

Here are some key points to remember about useMemo:

- **Stable References**: `useMemo` keeps stable references to the exact same object in memory, ensuring that strict equality (===) can be satisfied.
- **Dependency-Based Rerendering**: It reruns only when the dependencies in the array change.
- **Good Use Cases**: It's ideal for scenarios where you have an expensive operation that you don't want to re-run every time your component renders.
- **Prevents Unnecessary Re-renders**: Selectors, which are often used in state management libraries like Redux, should be memoized if they return objects or arrays. Memoizing selectors ensures that they are not recalculated needlessly, contributing to a more efficient application.


**Bad Example**: Performing Expensive Computation on Every Render

In this example, we have a component that renders a list of items. It performs an expensive computation, such as sorting the items, on every render. This is inefficient and can lead to performance issues, especially with a large number of items.

```jsx
// BadListComponent.js
import React from 'react';

function BadListComponent({ items }) {
  const sortedItems = items.sort(); // Expensive computation on every render
  console.log('ListComponent rendered');

  return (
    <ul>
      {sortedItems.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}

export default BadListComponent;
```

**Good Example**: Using useMemo to Optimize Expensive Computations

In the improved version, we use useMemo to memoize the sorted items, ensuring that the expensive computation is only performed when the items dependency changes. This significantly improves the performance of the component.

```jsx
// GoodListComponent.js
import React, { useMemo } from 'react';

function GoodListComponent({ items }) {
  const sortedItems = useMemo(() => {
    return items.sort(); // Expensive computation only when 'items' changes
  }, [items]); // Include 'items' as a dependency

  console.log('ListComponent rendered');

  return (
    <ul>
      {sortedItems.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}

export default GoodListComponent;
```


## Problem: Unnecessary Rerenders

In React, components are designed to re-render when their state or props change. While this is a fundamental aspect of React's reactivity, it can lead to unnecessary and costly rerenders in certain scenarios.

Consider a common situation where a parent component renders multiple child components. If the parent component creates a new callback function on every render and passes it as a prop to the children, the child components may rerender even when their data remains unchanged. This is not an ideal scenario in terms of performance.

### The Solution: useMemo and useCallback

Memoizing selectors ensures that they are not recalculated needlessly, contributing to a more efficient application.

Because the dependency of the `useEffect` is an object, the `useEffect` will always rerun because the new object is considered a different object in memory, even if the value for `userId` has not changed.

```jsx
import React, { useState, useEffect, useMemo } from 'react';

function UserProfile({ userId }) {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    // Fetch user data from an API based on the userId
    fetch(`https://api.example.com/users/${userId}`)
      .then((response) => response.json())
      .then((data) => setUserData(data));
  }, [userId]); // Because this dependency is an object, the useEffect will always rerun because the new object is considered a different object in memory.

  return (
    <div>
      <h1>User Profile</h1>
      {userData ? (
        <div>
          <p>Name: {userData.name}</p>
          <p>Email: {userData.email}</p>
        </div>
      ) : (
        <p>Loading user data...</p>
      )}
    </div>
  );
}
```

Here's a React component that uses useMemo to memoize a dependency for a useEffect to prevent unnecessary re-renders. Now the useEffect will only re-run if the value for `userId` changes.

```jsx
import React, { useState, useEffect, useMemo } from 'react';

function UserProfile({ userId }) {
  const [userData, setUserData] = useState(null);

  // Use useMemo to memoize the dependency for useEffect
  const effectDependency = useMemo(() => [userId], [userId]);

  useEffect(() => {
    // Fetch user data from an API based on the userId
    fetch(`https://api.example.com/users/${userId}`)
      .then((response) => response.json())
      .then((data) => setUserData(data));
  }, effectDependency); // Use the memoized dependency

  return (
    <div>
      <h1>User Profile</h1>
      {userData ? (
        <div>
          <p>Name: {userData.name}</p>
          <p>Email: {userData.email}</p>
        </div>
      ) : (
        <p>Loading user data...</p>
      )}
    </div>
  );
}
```
By memoizing the dependency with `useMemo`, we ensure that it remains the same between renders unless the actual userId changes. This prevents unnecessary API requests and re-renders when other unrelated state variables change, making the component more efficient.

We've seen how we can use `useMemo` to prevent unnecessary re-renders. Now what about `useCallback`?

`useCallback` is a React hook that memoizes functions. It ensures that a new function is created only when its dependencies change. This solves the problem of unnecessary rerenders, especially when passing functions as props to child components.

`useCallback` is `useMemo`, but with some syntactic sugar on top. It is specifically designed for functions. `useCallback` returns a memoized version of the function, ensuring that a new function is created only when its dependencies change. It provides a more concise way to memoize functions, so you don't need to return a function explicitly. See the example of `useMemo` and `useCallback` that memoize the same function for debouncing:

```jsx
// useMemo
const debouncedOnChange = React.useMemo(() => {
    return debounce(
    (e: React.ChangeEvent<HTMLInputElement>) => onChange?.(e), 300);
}, [onChange]);
```

```jsx
// useCallback
const debouncedOnChange = useCallback(
  debounce((e: React.ChangeEvent<HTMLInputElement>) => onChange?.(e), 300),
  [onChange]
);
```

Notice how they are almost the same, and you can use either.

Here are some key points to remember about `useCallback`:

- **Function Memoization**: It's used to memoize functions instead of objects or arrays.
- **Child Component Performance**: When passing a function as a prop to a child component, consider using useCallback. This is because, without it, the child component will re-render every time the parent component renders, which can be a problem if the child component is slow or has side effects you want to avoid.
- **Debouncing**: A common use case for useCallback is debouncing. You can use it to create a memoized function that only changes if certain other events or changes occur. This prevents a flood of callbacks firing when you want to limit the frequency of updates.

**Bad Example**: Unnecessarily Rerendering Components

In this example, we have a parent component that renders a list of child components. Each child component receives a callback function as a prop. In the parent component, the callback function is created on every render. This results in unnecessary rerenders of child components, even when their data hasn't changed.

```jsx
// BadParentComponent.js
import React, { useState } from 'react';

function ChildComponent({ onClick }) {
  console.log('ChildComponent rendered');
  return <button onClick={onClick}>Click me</button>;
}

function BadParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

export default BadParentComponent;
```

**Good Example**: Using useCallback to Prevent Unnecessary Rerenders

In the improved version, we use useCallback to memoize the handleClick function. This ensures that the function is not recreated on each render, preventing unnecessary rerenders of child components.

```jsx
// GoodParentComponent.js
import React, { useState, useCallback } from 'react';

function ChildComponent({ onClick }) {
  console.log('ChildComponent rendered');
  return <button onClick={onClick}>Click me</button>;
}

function GoodParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]); // Include 'count' as a dependency

  return (
    <div>
      <p>Count: {count}</p>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

export default GoodParentComponent;
```

In React applications, performance optimization is a critical consideration as your projects grow. `useMemo` and `useCallback` are valuable tools that address the problems of unnecessary rerenders and expensive computations, helping you create more efficient and responsive applications. By applying these hooks effectively, you can provide a smoother user experience while maintaining the simplicity and elegance of React development.
