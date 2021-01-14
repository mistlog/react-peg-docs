---
id: rule
title: Rule
---

Traditionally, a grammar is a list of rules and you must define the grammar explicitly. In React Peg, we only care about rules and you can treat it as UI components.

## Render

The result of rendering a rule is a plain object just like that in React, and this object describes what we want to parse against.

```javascript
function Text() {
    return (
        <text>abc</text>
    )
}

const text = new (<Text/>).rule({});
console.log(text);
```
and the result will be:

```javascript
{
    "props": {},
    "children": [
        "abc"
    ],
    "tagName": "text"
}
```

In this example, ```Text``` means that we want to parse(match) literal string ```"abc"```. 

## Action

Action method is the place where you harvest, you may want to use ```globalFunction.text()``` to get the matched text.

```js
function Text() {
    const action = ({globalFunction})=> {
        return {
            text: globalFunction.text()
        }
    }; 
    return (
        <pattern action={action}>
            <text>abc</text>
        </pattern>
    )
}

const parser = ReactPeg.render(<Text />);
const ast = parser.parse("abc");
console.log(ast);
```

The result will be:

```js
{
    "text": "abc"
}
```

## Built-In Rules

Built-In rules are the basic building blocks in React Peg, they are:

* text: matches literal string
* repeat: one or more(+), zero or more(*)
* list: play the role of \<div/> in html and Fragment in React
* set: just like \[a-zA-Z0-9]
* opt: optional(?)
* assert: assert exist/not exist, or failed
* or: matches a or b
* pattern: specify action

### text

In previous example, you can log ```parser.grammar``` to find out what happens behind the scene:

```javascript
console.log(parser.grammar);
```

The grammar under the hood:

```javascript
{
    const { actions } = options;
    const globalFunction = {
        location,
        text
    };
}
        

Text = ('abc' { 
    return actions.get("action0")({globalFunction});
})
```

In fact, React Peg is not a parser generator, [PEG.js](https://pegjs.org/documentation) is, what React Peg does is translating JSX to grammar text and injecting actions.

The explanation of ```text``` adapted from PEG.js:

> Match exact literal string

### repeat

You must specify ```type``` in props to use ```repeat``` rule:

> "*": Match zero or more repetitions of the expression

> "+": Match one or more repetitions of the expression

### list

> Match a sequence of expressions

You can use it like \<div/> to group a bunch of rules.

### set

> Match one character from a set

For example, you can render ```Word``` as:

```javascript
<set>A-Za-z0-9_</set>
```

By the way, you can always log ```parser.grammar``` to inspect the grammar if you are in doubt about the rendered rule.

### opt

> Try to match the expression. If the match succeeds, return match result, otherwise return null.

### assert

You must specify ```type``` in props to use ```assert``` rule:

> with: Try to match the expression. If the match succeeds, just return undefined and do not consume any input, otherwise consider the match failed.

> without: Try to match the expression. If the match does not succeed, just return undefined and do not consume any input, otherwise consider the match failed.

For example:

```javascript
function AssertTest () {
    const action = ({globalFunction})=> {
        return {
            text: globalFunction.text()
        }
    }; 
    return (
        <list>
            <text>a</text>
            <assert type="with">
                <text>c</text>
            </assert>
            <text>c</text>
        </list>
    )
}

const parser = ReactPeg.render(<AssertTest/>);
const ast =  parser.parse(`ac`);
console.log(ast);
```

The result will be:

```javascript
[ 'a', undefined, 'c' ]
```

If assert failed, ie. what follows 'a' isn't 'c', the match will fail and abort.

Change 'c' to 'd' in ```parser.parse('ac')``` and have a try, you will get:

```javascript
SyntaxError: Expected , or undefined but "a" found.
...etc
```

It seems like some kind of lookahead.

### or

> Try to match the first expression, if it does not succeed, try the second one, etc.

### pattern

Wrap rules in pattern and specify action, you can get any part of matched result:

> label will be used as param in action

```js
function Text() {
    const action = ({part1, part2})=> {
        return {
            part1,
            part2
        }
    }; 
    return (
        <pattern action={action}>
            <text label="part1">abc</text>
            <text label="part2">def</text>
        </pattern>
    )
}

const parser = ReactPeg.render(<Text />);
const ast = parser.parse("abcdef");
console.log(ast);
```

the result will be:

```js
{
    "part1": "abc",
    "part2": "def"
}
```