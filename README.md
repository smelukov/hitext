<img align="right" width="125" height="125"
     alt="HiText logo"
     src="https://user-images.githubusercontent.com/270491/41946489-795b7e6a-79bb-11e8-9b1f-012b0dee3f0a.png"/>

# HiText

[![NPM version](https://img.shields.io/npm/v/hitext.svg)](https://www.npmjs.com/package/hitext)
[![Build Status](https://travis-ci.org/hitext/hitext.svg?branch=master)](https://travis-ci.org/hitext/hitext)
[![Coverage Status](https://coveralls.io/repos/github/hitext/hitext/badge.svg?branch=master)](https://coveralls.io/github/hitext/hitext?branch=master)

HiText is a basis for a text (source code) decoration. The main goal is to provide a universal way to combine libraries decorating text and an output in the required format (HTML, TTY etc). The only prerequsites you need are a source text and set of ranges with some options, the rest will be done by HiText.

## Why?

Just imagine you want to output in HTML a JavaScript code with syntax highlighting and to spotlight some fragments of it. You can use a library for syntax highlighting, that's not a problem. For fragment spotlighting you can get substrings and wrap them into HTML tags. Both operations are not so hard, but the problem is to combine results of them. Each operation adds HTML tags to a result, so another operation will not perform as expected because of HTML tags.

The solution is to change such operations to produce a set of ranges instead of markup. Once ranges is generated, HiText can merge them by universal rules and get a stream of segments. As a final step, segment stream is used to generate open and close tags when needed.

The approach allows to combine any number of decorators (which became range generators). Moreover, using a source and a range set, it's possible to output a result in any format beside HTML. An output format is defined by a printer.

## Features

- Univeral way to combine any number of decorators (generators)
- Much easier to make a generator
- Parser with formatting loses can be used. No necessary to mutate AST to decorate a code
- Flexible setup for output format. Build-in printers: HTML, TTY (terminal)

## Example

```js
const hitext = require('hitext');
const prism = require('hitext-prism');
const printer = hitext.printer.html.fork(prism.printer.html);
const source = 'const a = 1;\nconst b = 2;';

const generators = [
    prism('js'),
    hitext.generator.spotlight([6, 11], [19, 24])
];

console.log(
    hitext.decorate(source, generators, printer)
);

// or

const decorator = hitext()
    .use(prism('js'))
    .use(hitext.generator.spotlight([6, 11], [19, 24]))
    .printer(printer);

console.log(decorator.decorate(source));
```

Output:

```html
<div><span class="token keyword">const</span> <span class="spotlight">a <span class="token operator">=</span> <span class="token number">1</span></span><span class="token punctuation">;</span>
<span class="token keyword">const</span> <span class="spotlight">b <span class="token operator">=</span> <span class="token number">2</span></span><span class="token punctuation">;</span></div>
```

![image](https://user-images.githubusercontent.com/270491/41946250-0df745e2-79ba-11e8-8b32-38a9f938a380.png)

## Build-in generators

### spotlight(...ranges)

```js
const hitext = require('hitext');

console.log(
    hitext.decorate('1234567890', [hitext.generator.spotlight([3, 6])], 'html')
);
// '<div>123<span class="spotlight">456</span>7890</div>'
```

### match(pattern, match)

```js
const hitext = require('hitext');

console.log(
    hitext.decorate('Hello world! Hello world!', [hitext.generator.match('world')], 'html')
);
// <div>Hello <span class="match">world</span>! Hello <span class="match">world</span>!</div>

console.log(
    hitext.decorate('Hello world!', [hitext.generator.match(/\w+/)], 'html')
);
// <div><span class="match">Hello</span> <span class="match">world</span>!</div>

console.log(
    hitext.decorate('Hello world!', [hitext.generator.match(/\w+/, 'spotlight')], 'html')
);
// <div><span class="spotlight">Hello</span> <span class="spotlight">world</span>!</div>
```

## License

MIT
