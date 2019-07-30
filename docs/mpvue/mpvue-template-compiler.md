---
sidebar: true
prev: false
next: false
---

# mpvue-template-compiler

> This package is auto-generated. For pull requests please see [src/platforms/mp/entry-compiler.js](https://github.com/Meituan-Dianping/mpvue/blob/master/src/platforms/mp/entry-compiler.js).

This package can be used to pre-compile Vue 2.0 templates into render functions to avoid runtime-compilation overhead and CSP restrictions. You will only need it if you are writing build tools with very specific needs. In most cases you should be using [mpvue-loader](http://mpvue.com/build-tool/mpvue-loader/) instead, both of which use this package internally.

## Installation

``` bash
npm install mpvue-template-compiler
```

``` js
const compiler = require('mpvue-template-compiler')
```

## API

### compiler.compile(template, [options])

Compiles a template string and returns compiled JavaScript code. The returned result is an object of the following format:

``` js
{
  ast: ?ASTElement, // parsed template elements to AST
  render: string, // main render function code
  staticRenderFns: Array<string>, // render code for static sub trees, if any
  errors: Array<string> // template syntax errors, if any
}
```

Note the returned function code uses `with` and thus cannot be used in strict mode code.

#### Options

It's possible to hook into the compilation process to support custom template features. **However, beware that by injecting custom compile-time modules, your templates will not work with other build tools built on standard built-in modules, e.g `mpvue-loader` and `vueify`.**

The optional `options` object can contain the following:

- `modules`

  An array of compiler modules. For details on compiler modules, refer to the `ModuleOptions` type in [flow declarations](https://github.com/vuejs/vue/blob/dev/flow/compiler.js) and the [built-in modules](https://github.com/vuejs/vue/tree/dev/src/platforms/web/compiler/modules).

- `directives`

  An object where the key is the directive name and the value is a function that transforms an template AST node. For example:

  ``` js
  compiler.compile('<div v-test></div>', {
    directives: {
      test (node, directiveMeta) {
        // transform node based on directiveMeta
      }
  })
  ```

  By default, a compile-time directive will extract the directive and the directive will not be present at runtime. If you want the directive to also be handled by a runtime definition, return `true` in the transform function.

  Refer to the implementation of some [built-in compile-time directives](https://github.com/vuejs/vue/tree/dev/src/platforms/web/compiler/directives).

- `preserveWhitespace`

  Defaults to `true`. This means the compiled render function respects all the whitespaces between HTML tags. If set to `false`, all whitespaces between tags will be ignored. This can result in slightly better performance but may affect layout for inline elements.

---

### compiler.compileToFunctions(template)

Similar to `compiler.compile`, but directly returns instantiated functions:

``` js
{
  render: Function,
  staticRenderFns: Array<Function>
}
```

This is only useful at runtime with pre-configured builds, so it doesn't accept any compile-time options. In addition, this method uses `new Function()` so it is not CSP-compliant.

---

### compiler.ssrCompile(template, [options])

> 2.4.0+

Same as `compiler.compile` but generates SSR-specific render function code by optimizing parts of the template into string concatenation in order to improve SSR performance.

This is used by default in `vue-loader@>=12` and can be disabled using the [optimizeSSR](https://vue-loader.vuejs.org/en/options.html#optimizessr) option.

---

### compiler.ssrCompileToFunction(template)

> 2.4.0+

Same as `compiler.compileToFunction` but generates SSR-specific render function code by optimizing parts of the template into string concatenation in order to improve SSR performance.

---

### compiler.parseComponent(file, [options])

Parse a SFC (single-file component, or `*.vue` file) into a descriptor (refer to the `SFCDescriptor` type in [flow declarations](https://github.com/vuejs/vue/blob/dev/flow/compiler.js)). This is used in SFC build tools like `vue-loader` and `vueify`.

#### Options

#### `pad`

`pad` is useful when you are piping the extracted content into other pre-processors, as you will get correct line numbers or character indices if there are any syntax errors.

- with `{ pad: "line" }`, the extracted content for each block will be prefixed with one newline for each line in the leading content from the original file to ensure that the line numbers align with the original file.
- with `{ pad: "space" }`, the extracted content for each block will be prefixed with one space for each character in the leading content from the original file to ensure that the character count remains the same as the original file.

---

### compiler.compileToWxml(compiled, [options])
Parse a AST（Abstract Syntax Tree）into a String which is the type of string of a WXML(WeiXin Markup Language) file.This is used after a template compiled into a AST with the function `compiler.compile(template, [options])` .

#### compiled
`compiled` is the result of [compiler.compile](#compilercompiletemplate-options),with this function you can compile the [template](#https://cn.vuejs.org/v2/guide/syntax.html) strings.

#### Options
- components: `components` is an Object includes the properties of the components in the template where is the paramter `compiled` comes from.
`components` is necessary if there are components declared in your vue file before compiling. The formate of `components` maybe like this:
 `{
 componentA: { src: '/components/coma', name: 'componentA' },
 componentB: { src: '/components/comb', name: 'componentB' }
 }`
- moduleId: The unique identifier of component.Using `moduleId` you can inject an only name of components and when you use it in `[css scoped]` or `[css modules]` ,the style of component won't be covered.

