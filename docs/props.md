---
id: props
title: Props
---

You can pass props to a rule and specify the interface of props in the type parameter in ```ReactPeg.Rule``` like this:

```javascript
function Text (props) {
    console.log(props);
    return (
        <text>{`ab${props.text}`}</text>
    );
}

const parser = ReactPeg.render(<Text foo="foo" text="c"/>);
const ast = parser.parse(`abc`);
console.log(ast);
```

the result will be:

```javascript
{ foo: 'foo', text: 'c' }
abc
```