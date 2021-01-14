---
id: getting-started
title: Getting Started
sidebar_label: Getting Started
---

## About React Peg

This project is inspired by React and PEG.js, as you can see from the icon and style of this site. However it's not deliberate, and it's really hard to design and build a site from scratch.

React Peg is designed to generate parser with confidence. PEG.js is easy to use but lack of type checking and reuse mechanism. JSX is suitable for nested structures and the concept of render in React is extended here to target grammar.

Refer to [PEG.js](https://pegjs.org/) for more information about peg.

## Playground

You can play with it online in [react-peg-playground](https://mistlog.github.io/react-peg-playground/).

## Installation

You can install React Peg via [npm](https://www.npmjs.com/package/react-peg):

```bash
 > npm i react-peg
```

or use template project: [react-peg-template](https://github.com/mistlog/react-peg-template)

```js
npx degit mistlog/react-peg-template#main react-peg-template
cd react-peg-template
npm install 
npm run example:calculator
```

## Building your first Rule

In the template project, open [number.tsx](https://github.com/mistlog/react-peg-template/blob/main/src/number.tsx):

```js
import { ReactPeg } from "react-peg";
export function Number() {
    const digits = (
        <repeat type="+">
            <set>0-9</set>
        </repeat>
    );

    const action = ({ globalFunction }) => {
        return parseFloat(globalFunction.text());
    }

    return (
        <pattern action={action}>
            {digits}
            <opt>
                <text>.</text>
                {digits}
            </opt>
        </pattern>
    );
}
```

This simple example shows you how to define a rule and perform action while parsing. Open [number.test.tsx](https://github.com/mistlog/react-peg-template/blob/main/src/number.test.tsx):

```javascript
import { ReactPeg } from "react-peg";
import { Number } from "./number";

test("parse number", () => {
    const parser = ReactPeg.render(<Number />);
    const ast = parser.parse("3.14");
    expect(ast).toEqual(3.14);
})
```

You can figure out the pattern here, we define grammar rule in function method with the help of JSX to be expressive about the nested structure.

```ReactPeg.render``` takes a rule and give you a parser, the rule passed will be the start rule of your grammar, and all related rules will be collected silently while "rendering" the start rule, you don't need to define the grammar as a list of rules explicitly.

You can find more information about JSX [here](https://reactjs.org/docs/introducing-jsx.html).