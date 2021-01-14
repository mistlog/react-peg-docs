---
id: calculator
title: Calculator
---

In this example, we will build a simple arithmetic calculator which takes input such as:

```javascript
1.5 + 2 + 3.4 * ( 25 - 4 ) / 2 - 8
```

## Step 1: Factor

> Factor = Expression or Number

Factor can be expression or number, however, we will parse it as Number only in this step.

Let's define Number rule:

```javascript
function Number() {
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

then, Factor rule:

```javascript
class Factor extends ReactPeg.Rule<{ label?: string }, number>{

    render()
    {
        return (
            <list label="factor">
                <_ />
                    <Number />
                <_ />
             </list>
        );
    }

    action(param: FactorActionParam)
    {
        const { factor } = param;
        const [value] = factor.filter(item => item !== null && typeof item !== "string") as [number];
        return value;
    }
}
```

to ignore white spcae, we need to define:

```javascript
class _ extends ReactPeg.Rule<{}, null>{

    render()
    {
        return (
            <repeat type="*">
                <set> \t\n\r</set>
            </repeat>
        );
    }

    action(param: ActionParam)
    {
        return null;
    }
}
```

then log the result:

```javascirpt 
const parser = ReactPeg.render(<Factor />);
const ast = parser.parse("12.34");
console.log(ast);
```

```javascript
12.34
```

type declaration is removed for readability, see complete program in this [gist](https://gist.github.com/mistlog/16edf6b8cd1e07d2ca8f78e2e0e17a4b).

## Step 2: Term

> Term = Factor ( * or / Fcator) *

Then, let's move on and define term:

```javascript 
class Term extends ReactPeg.Rule<{ label?: string }, TermNode>{

    render()
    {
        const head = (
            <Factor label="head" />
        );

        const tail = (
            <repeat label="tail" type="*">
                <_ />
                <or>
                    <text>*</text>
                    <text>/</text>
                </or>
                <Factor />
                <_ />
            </repeat>
        );

        return (
            <list>
                {head}
                {tail}
            </list>
        );
    }

    action(param: TermActionParam)
    {
        const { head, tail } = param;

        const value = this.prettifyTail(tail).reduce((prev: number, curr: [TermOpType, FactorNode]) =>
        {
            const [op, factor] = curr;

            if (op === "*") return prev * factor;
            if (op === "/") return prev / factor;

        }, head);

        return value;
    }

    prettifyTail(tail: TermTailNode)
    {
        return tail.map(
            item => item.filter(e => e !== null)
        );
    }
}
```

Now, we can evaluate input such as ```12.34 * 5```:

```javascript
const parser = ReactPeg.render(<Term />);
const ast = parser.parse("12.34 * 5");
console.log(ast);
```

```javascript
61.7
```

see complete program in this step in [gist](https://gist.github.com/mistlog/fde75c85167870905f52c4d411eec783).

## Step 3: Expression

> Expression = Term ( + or - Term) *

Then, the final piece of our simple calculator:

```javascript
class Expression extends ReactPeg.Rule<{}, ExpressionNode>{

    render()
    {
        const head = (
            <Term label="head" />
        );

        const tail = (
            <repeat label="tail" type="*">
                <_ />
                <or>
                    <text>+</text>
                    <text>-</text>
                </or>
                <Term />
                <_ />
            </repeat>
        );

        return (
            <list>
                {head}
                {tail}
            </list>
        );
    }

    action(param: ExpressionActionParam)
    {
        const { head, tail } = param;

        const value = this.prettifyTail(tail).reduce((prev: number, curr: [ExpressionOpType, TermNode]) =>
        {
            const [op, factor] = curr;

            if (op === "+") return prev + factor;
            if (op === "-") return prev - factor;

        }, head);

        return value;
    }

    prettifyTail(tail: ExpressionTailNode)
    {
        return tail.map(
            item => item.filter(e => e !== null)
        );
    }
}
```

don't forget to add Expression to the Factor rule:

```javascript
class Factor extends ReactPeg.Rule<{ label?: string }, FactorNode>{

    render()
    {
        const number = (
            <list>
                <_ />
                <Number />
                <_ />
            </list>
        );

        const expression = (
            <list>
                <_ />
                <text>(</text>
                <Expression />
                <text>)</text>
                <_ />
            </list>
        );

        return (
            <or label="factor">
                {expression}
                {number}
            </or>
        );
    }

    action(param: FactorActionParam)
    {
        const { factor } = param;
        const [value] = factor.filter(item => item !== null && typeof item !== "string") as [number];
        return value;
    }
} 
```

That's all!

```javascript
const parser = ReactPeg.render(<Expression />);
const ast = parser.parse("1.5 + 2 + 3.4 * ( 25 - 4 ) / 2 - 8");
console.log(ast);
```

```javascript
31.199999999999996
```

see complete program in this [gist](https://gist.github.com/mistlog/4cfdc0f3f913f797724f9744f020e7ab).