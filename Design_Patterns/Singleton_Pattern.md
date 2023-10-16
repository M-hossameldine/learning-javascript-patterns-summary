# Singleton Pattern

Singletons are classes that can be instantiated once, and can be accessed globally. This single instance can be shared throughout our application, which makes Singletons great for managing global state in an application.

### Counter Application

[Live Demo on CodeSandbox](https://codesandbox.io/embed/lucid-morning-64mr1)

We're going to build a counter class that has

- **getInstance** method that returns the value of the instance
- **getCount** method that returns the current value of the counter variable
- **increment** method that increments the value of counter by one
- **decrement** method that decrements the value of counter by one

To Match Singleton Requirements

- Use **instance** variable to make sure that Counter class to be only **instantiated once**, we can run this validation in the constructor of Counter, we can set **instance** value equal to a reference to the instance when a new instance is created. We can prevent new instantiations by checking if the **instance** variable already had a value;
- You should freeze the instance as well. The **Object.freeze** method makes sure that consuming code cannot modify the Singleton. Properties on the frozen instance cannot be added or modified, which reduces the risk of accidentally overwriting the values of the Singleton.

```
counter.js

let instance;
let counter = 0;

class Counter {
  constructor() {
    if(instance) {
      throw new Error("You can only have one instance!");
    }
    instance = this;
  }

  geInstance() {
    return this;
  }

  getCount() {
    return counter;
  }

  increment() {
    return ++counter;
  }

  decrement() {
    return --counter;
  }

  const singletonCounter = Object.freeze(new Counter());
  export default singletonCounter;
}
```

Both blueButton.js and redButton.js import **the same instance** from counter.js

redButton.js

```
import Counter from './counter';

const button = document.getElementById ('red');
button.addEventListener('click', () => {
  Counter.increment();
  console.log("Counter total: ", Counter.getCount());
})
```

blueButton.js

```
import Counter from './counter';

const button = document.getElementById ('blue');
button.addEventListener('click', () => {
  Counter.increment();
  console.log("Counter total: ", Counter.getCount());
})
```

### Tradeoffs

- Singleton is considered **anti-pattern**, and can (or.. should) be avoided in JavaScript.
- I languages such Java or C++, it's not possible to directly create objects the way we can in JavaScript. In those object-oriented programming languages, we need to create a class, which creates an object. That created object has the value of the instance ot the class. just like the value of **instance** in the JavaScript example.

#### Using a regular object

Same Counter example but the **counter** is simply an object

[Sandbox](https://codesandbox.io/embed/competent-moon-rvzrr)

```
let count= 0

const counter = {
  increment () {
    return ++count;
  },
  decrement () {
    return --count;
  }
}

Object.freeze(counter);

export {counter}
```

Since objects are passed by reference, both redButton.js and blueButton.js are importing a reference to the same counter object. Modifying the value of **count** in either of these files will modify the value on the counter, which is visible in both files.

#### Testing

Testing code that relies on singleton can get tricky. Since we cant't create new instance each time, all tests rely on the modification to the global instance of the previous test.

- The order of testing matters in this case, and one small modification can lead to an entire test suite failing.
- After testing, we need to reset the entire instance in order to reset modifications made by the tests.

[Sandbox](https://codesandbox.io/embed/sweet-cache-n55vi)

```
import Counter from "../src/counterTest";

test("increment 1 time should be 1", () => {
  Counter.increment();
  expect(Counter.getCount()).toBe(1);
})

test("incrementing 3 extra times should be 4", () => {
  counter.increment();
  counter.increment();
  counter.increment();
  expect(Counter.getCount()).toBe(4);
})

test("decrementing 1 times should be 3", () => {
  Counter.decrement();
  expect(Counter.getCount().toBe(3));
} )
```

#### Dependency hiding

When importing another module, **superCounter.js** in this case, it may not be obvious that module is importing a Singleton. in other files, such as **index.js** in this case, we may be importing that module and invoke its methods. This way, we accidentally modify the values in the Singleton. This can lead to unexpected behavior, since multiple instances for the Singleton can be shared throughout the application.

[Sandbox](https://codesandbox.io/embed/sweet-cache-n55vi)

```
import Counter from "./counter";

export default class SuperCounter {
  constructor() {
    this.count =0;
  }

  increment() {
    Counter.increment();
    return (this.counter += 100);
  }

  decrement() {
    Counter.decrement();
    return(this.count -= 100);
  }
}
```

### Global Behavior

- A Singleton instance should be able to get referenced throughout the entire app.
- Having global variables is generally considered as a bad design decision. Global scope pollution can end up in accidentally overwriting the value of a global variable. which can lead to a lot of unexpected behavior.

#### In ES2015

- The new **let** and **const** keyword are **block-scoped** to prevent developers from accidentally polluting the global scope
- The new **module** system in JavaScript makes creating globally accessible values easier without polluting the global scope, by being able to **export** values from a module, and **import** those values in other files.

#### Order of Execution & Data Flow

- The order of execution is important: we don't want to accidentally consume data first, when there is not data to consume (yet).
- Understanding the data flow when using a global state can get very tricky as you application grows, and dozens of components rely on each other.

### State management in React

We often rely on a global state through state management tools such as **Redux** or **React Context** instead of using Singletons. Although their global state behavior might seem similar to that of a Singleton.

Those tools provide a **read-only state** rather than the **mutable state** of the Singleton. When using Redux, only pure function reducers can update the state, after a component has sent an action through a dispatcher. so you make sure that the global state is mutated the way we intended it, since components cannot update the state directly.
