# JS Style Guide

## Code for Others 

- Your code is going to be read many, many, many other developers. Always be conscious and thoughtful about making their jobs as easy as possible.  
- Code is craft. Be proud of the code your write. If you aren't, get feedback and ideas from others!  

## Type System
- All files must be typed 
- Use Typescript or [flow](https://flow.org/en/)

## Follow Functional Paradigms

- Functions should be pure and maintain [referential transparency](https://www.sitepoint.com/what-is-referential-transparency/)
- Functions should not have unnecessary side effects
- Functions must be unit testable
- [Leverage ES6 syntax](https://medium.com/dailyjs/functional-js-with-es6-recursive-patterns-b7d0813ef9e3) to make writing functional JS easier
- If want to use `.forEach`, then check [lodash](https://lodash.com/docs/4.17.11) for a more functional solution

## Avoid Mutation at All Costs!

- **Never** user `var` and **avoid** using `let` unless strictly needed.
- If you find yourself using `let`, check the following:
  1. Can I use a `ternary operator` for assignment?
     ```js
     const conditionalVariable = Boolean(otherVariable)
       ? 'Condition true'
       : 'Condition false';
     ```
  2. Can I extract this conditional logic into a helper function?
     ```js
     const multipleConditionsHelper = someValue => {
       switch (someValue) {
         case 'A':
           return 'Value A';
         case 'B':
           return 'Value B';
         default:
           return 'Value C';
       }
     };

     // further down in the file
     const otherFunction = otherValue => {
       const conditionalValue = multipleConditionsHelper(otherValue);
     };
     ```
- Never do `someObject.foo = 'a'`
    ```js
    // BAD
    const someObject = { bar: 'b' };
    someObject.foo = 'a';

    // GOOD
    const someObject = { bar: 'b' };
    const newObject = { ...someObject, foo: 'a' };

    // ALSO GOOD
    const someObject = {
      bar: 'b',
      foo: booleanValue ? 'a' : 'c',
    };

    // ALSO GOOD
    const someObject = {
      bar: 'b',
      ...(booleanValue ? { foo: 'a' } : {}),
    };
    ```
- If you are desparate for mutation, use [immer](https://github.com/mweststrate/immer). Immer is also great for simplifying [redux reducers](https://github.com/reduxjs/react-redux)!

## Variable Naming

- Variables and functions must be `camelCased` or uppercased `SNAKE_CASE` for global constants
- Never use lowercase `snake_case`

  ```js
  // GOOD
  const someFunction = () => {};
  const someVariable = [];
  const SOME_CONSTANT = 1337;

  // BAD
  const bad_variable_name = {};
  ```

- Variable names should signify its purpose
  ```js
  // GOOD
  const nameList = ['Devin', 'Joe', 'Josh'];
  // BAD
  const list = ['Devin', 'Joe', 'Josh'];
  ```
- No magic numbers. Use constants for numbers.
  ```js
  // GOOD
  const POLL_RATE = 15 * 1000;

  const pollIntervalId = setInterval(() => {
    console.log('poll');
  }, POLL_RATE);

  // BAD
  const pollIntervalId = setInterval(() => {
    console.log('poll');
  }, 15 * 1000);
  ```

## Follow DRY Principles

- If you find yourself duplicating logic extract, it into a utility function
- If you repeat logic twice it can be OK, but duplicating logic three times is `rarely` acceptable
- If you create a helper function to simplify the logic in a function, do not define it within the scope of the function!
  ```js
  // GOOD
  const extractedComplicatedLogic = value => {
    // do stuff
  };

  const importantFunction = currentValue => {
    const updatedValue = extractedComplicatedLogic(currentValue);
    // do more stuff
  };

  // BAD
  const importantFunction = currentValue => {
    const extractedComplicatedLogic = value => {
      // do stuff
    };

    const updatedValue = extractedComplicatedLogic(currentValue);
    // do more stuff
  };
  ```
 - Always keep abstractions in mind.
   - How can I make this a repeatable pattern?
   - How can I make a helper to make this abstract away boilerplate?
   - If you think of an abstraction idea and don't have time, then leave a `TODO`!

## Comments for Clarity

- Comments should enhance the readability of your code
- Always comment when understanding your code requires context beyond the scope of the file
  - Explain the business logic that drove your decision
  - Warn future developers that changing certain code will have unintended behaviors
  - Emphasize the importance of a piece of code
- If you don't love your variable name, its probably a bad variable name ☹️. But if you can't come up with a better one, leave a comment to explain the purpose of the variable!
- Comment to help with future refactors
- It's way harder to comment too much than it is to comment too little, so when in doubt, leave a comment!

## Use Guards over Nested If/Else 💂‍♀️

- Guards make functions more declarative and readable!
- If you use an `else` statement, then check to see if you can re-write using a guard.
  ```js
  // BAD
  const someFunction = someValue => {
    if (someValue) {
      return 'A';
    } else {
      return 'B';
    }
  };

  // GOOD
  // Much more readable!
  const someFunction = someValue => {
    if (!someValue) return 'B';

    return 'A';
  };
  ```

## Always Favor ES6 Module Imports and Exports. Avoid Closures and Classes.

- Using `ES6` `import`/`export` makes it easier to unit test shared logic
- Poorly maintained classes and closures lose the benefits of tree-shaking and dead code elimination
- Avoiding classes and closures forces you to think functionally and reduces reliance on an internal state
- There are use cases for classes and closures, but do not use unless strictly necessary.
  ```js
  // GOOD

  // util.js
  export const logger = value => console.log(`Logging ${value}`);

  // other-file.js
  import { logger } from './util.js';

  // ------------------------------------

  // GOOD
  // logger.js
  const logger = value => console.log(`Logging ${value}`);

  export default logger;

  // other-file.js
  import logger from './logger';

  // ------------------------------------
  // BAD
  // util.js
  class Utilities {
    logger = () => console.log(`Logging ${value}`);
  }

  export default Utilities;

  // other-file.js
  import Utilities from './util';
  ```

## Use Named Parameters (Objects) for Shared Functions

- Use objects parameters for `import`-ed or shared functions
- Order doesn't matter! The beauty of using objects for parameters, means future developers do not have to worry parameter ordering!

  ```js
  // GOOD

  // helper.js
  const helperFunction = ({ foo, bar }) => return `${foo}${bar}`;

  // some-file.js
  import { helperFunction } from "../helpers";

  // YAY! Other developers do not need to worry about order, only parameter names.
  const data = helperFunction({ bar: 'b', foo: 'a' });

  // ------------------------------------------------------------------------
  // BAD
  const badHelper = (foo, bar) => return `${foo}${bar}`

  // (BAD) Some developer accidentally messes up the order without knowing!
  const localFoo = 'a';
  const localBar = 'b';
  const data = badHelper(localBar, localFoo);
  ```
  
## ❤️ ES6 Arrow Functions , Destructuring, and Spreads

- Always use arrow functions for consistencey
  ```js
  // GOOD
  const someFunction = () => {};

  // BAD
  function someFunction() {}
  ```
- Destructuring for object and array parameters/variables
- Destructure for variable assignment
  ```js
  // GOOD

  // Ex. Simple destructing parameters
  const functionWithObject = ({ foo, bar }) => {};
  // Ex. Rename parameters
  const functionWithObject = ({ foo: renamedFoo, bar: renamedBar }) => {};

  const assignmentWithDestructing = person => {
    const { firstName, lastName } = person;
  };

  // BAD
  const assignmentWithoutDestructiong = person => {
    const firstName = person.firstName;
    const lastName = person.lastName;
  };
  ```
- Use the spread operation `...` for merging and cloning objects and manipulating arrays!
  ```js
  const mergedObject = { ...oneObject, ...secondObject };
  const clonedObject = { ...mergedObject };

  const concatArrays = (arrayOne, arrayTwo) => [...arrayOne, ...arrayTwo];
  const appendToArray = (array, item) => [...array, item];
  const head = ([firstItem]) => firstItem;
  const [firstItem, secondItem] = veryLongArray;
  ```
- Avoid creating uneccesary function
  - Most importantly for `react` component props
  - Use `_.flow` when you can to create a function that chains multiple functions

## Git Guidelines

- Name branches `username/short-branch-description`
- Always have good [git hygiene](http://www.ericbmerritt.com/2011/09/21/commit-hygiene-and-git.html)
- **Do not commit commented code**

## ESLint

```js
module.exports = {
  env: {
    browser: true,
    'jest/globals': true,
  },
  globals: {
    browser: true,
  },
  extends: [
    'airbnb', // tons of good defaults (very strict)
    'plugin:flowtype/recommended', // optional for flow!
    'plugin:jest/recommended',
    'prettier', // disable all rules that conflict with prettier
    'prettier/react', // disable all rules that conflict with prettier
    'prettier/flowtype', // disable all rules that conflict with prettier
  ],
  // only include flow-type if we are using flow
  plugins: ['flowtype', 'jest', 'react-hooks'],
  rules: {
    // START: Only if using flow
    'react/prop-types': 'off',
    'react/default-props-match-prop-types': 'off',
    'react/no-multi-comp': 'off',
    'react/require-default-props': 'off',

    // require "// flow" at the top of all js files
    'flowtype/require-valid-file-annotation': [
      2,
      'always',
      {
        annotationStyle: 'line',
      },
    ],
    // STOP: Only if using flow

    // Hooks :)
    'react-hooks/rules-of-hooks': 'error',

    'no-plusplus': 'off',
    'import/prefer-default-export': 'off',

    // airbnb rules that are too picky
    'react/destructuring-assignment': 'off',

    'lines-between-class-members': 'off', // too strict

    // we import types from jest
    'jest/no-jest-import': 'off',

    'no-unused-vars': [
      'error',
      {
        argsIgnorePattern: '^_',
      },
    ],
  },
};
```

## Prettier

```json
{
  "trailingComma": "all",
  "singleQuote": true,
  "semicolons": true
}
```

- Always format on save!

## Stylelint

- Assumes [Sass](https://sass-lang.com/guide)

```json
{
  "plugins": ["stylelint-scss"],
  "extends": "stylelint-config-recommended",
  "rules": {
    "at-rule-no-unknown": [
      true,
      {
        "ignoreAtRules": ["extend", "if"]
      }
    ],
    "selector-pseudo-class-no-unknown": [
      true,
      {
        "ignorePseudoClasses": ["global"]
      }
    ]
  }
}
```
