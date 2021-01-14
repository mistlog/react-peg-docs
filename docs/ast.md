---
id: ast
title: AST Node
---

Like props, you can specify the return type of ```action``` method in this way:

```javascript
import { ReactPeg, ActionParam } from "react-peg";

interface TextProps
{
    foo: string;
    text: string;
}

interface TextNode
{
    id: number;
    text: string;
}

class Text extends ReactPeg.Rule<TextProps, TextNode>{

    render()
    {
        return (
            <text>{`ab${this.props.text}`}</text>
        );
    }

    action(param: ActionParam)
    {
        return {
            id: 1,
            text: param.global.text()
        };
    }
}

const parser = ReactPeg.render(<Text foo="foo" text="c" />);
const ast = parser.parse(`abc`) as TextNode;
console.log(ast);
```

and the result will be:

```javascript
{ 
    id: 1, 
    text: 'abc' 
}
```