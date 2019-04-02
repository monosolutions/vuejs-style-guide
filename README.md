# Mono Solutions JavaScript Style Guide() {

<p align="center">
  <img style="max-width: 100px" src="img/logo.png"/>
</p>

*A mostly reasonable approach to VueJS*

Other Style Guides

  - [Javascript](https://github.com/monosolutions/javascript-style-guide)
  - [CSS & Sass](https://github.com/monosolutions/css-style-guide)

## Table of Contents

  1. [Component based development](#component-based-development)
  1. [Component structure](#component-structure)
  1. [Component naming](#component-naming)
  1. [Component data](#component-data)
  1. [v-for / v-if ](#vfor-vif)
  1. [Simple expressions in templates](#simple-expressions-templates)
  1. [Quoted attribute values](#quoted-attribute-values)
  1. [Directive shorthands](#directive-shorthands)
  1. [Implicit parent-child communication](#parent-child-communication)
  1. [Non-flux state manangement](#non-flux)
  1. [Testing](#testing)
  1. [ESLint](#eslint)
  1. [Resources](#resources)

## 1. Component based development
<a name="component-based-development"></a>

Always construct your app out of small components which do one thing and do it well.

A component is a small self-contained part of an application. The Vue.js library is specifically designed to help you create *view-logic components*.

### 1.1 Why?

Small components are easier to learn, understand, maintain, reuse and debug. Both by you and other developers.

### 1.2 How?

Each Vue component must be [FIRST](https://addyosmani.com/first/): *Focused* ([single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle)), *Independent*, *Reusable*, *Small* and *Testable*.

If your component does too much or gets too big, split it up into smaller components which each do just one thing. As a rule of thumb, try to keep each component file less than 100 lines of code.
Also ensure your Vue component works in isolation. For instance by adding a stand-alone demo.

**[⬆ back to top](#table-of-contents)**


## 2. Component structure
<a name="component-based-development"></a>

Using single file components ( SFC ) is prefered, as they promote modularity and the ease of development when working with components.

### 2.1 Order of top level elements

Single-file components should always have the `<template>`, `<script>`, and `<style>` order, with `<style>` last, because at least one of the other two is always necessary.

```html
//bad
<style>/* ... */</style>
<script>/* ... */</script>
<template>...</template>


//good
<template>...</template>
<script>/* ... */</script>
<style>/* ... */</style>
```

### 2.2 Component filename casing

Filenames of single-file components should either always be PascalCase.

PascalCase works best with autocompletion in code editors, as it’s consistent with how we reference components in JS(X) and templates, wherever possible.

```html
//bad
components/
|- mycomponent.vue

components/
|- myComponent.vue

//good
components/
|- MyComponent.vue

```

**[⬆ back to top](#table-of-contents)**

## 3. Component naming
<a name="component-based-development"></a>
> eslint: [`vue/name-property-casing`](https://eslint.vuejs.org/rules/name-property-casing.html#vue-name-property-casing)

Component names should be multi-word, except for root App components.

This prevents conflicts with existing and future HTML elements, since all HTML elements are a single word.

Each component name must be:

* **Meaningful**: not over specific, not overly abstract.
* **Short**: 2 or 3 words.
* **Pronounceable**: we want to be able to talk about them.

```javascript
//bad
export default {
  name: 'Todo',
  // ...
}

//good
export default {
  name: 'TodoItem',
  // ...
}
```

**[⬆ back to top](#table-of-contents)**

## 4. Component data
<a name="component-data"></a>

### 4.1 `data`
> eslint: [`vue/no-shared-component-data`](https://eslint.vuejs.org/rules/no-shared-component-data.html#vue-no-shared-component-data)

When using the `data` property on a component (i.e. anywhere except on new Vue), the value **must** be a function that returns an object. [More info](https://vuejs.org/v2/style-guide/#Component-data-essential)

```javascript
//bad
export default {
  data: {
    foo: 'bar'
  }
}

//good
export default {
  data () {
    return {
      foo: 'bar'
    }
  }
}
```

**[⬆ back to top](#table-of-contents)**

### 4.2 `props`
> eslint:
> * [`vue/require-prop-type-constructor`](https://eslint.vuejs.org/rules/require-prop-type-constructor.html#vue-require-prop-type-constructor)
> * [`vue/require-valid-default-prop`](https://eslint.vuejs.org/rules/require-valid-default-prop.html#vue-require-valid-default-prop)

**Prop definitions should be as detailed as possible.**

In committed code, prop definitions should always be as detailed as possible, specifying at least type(s) and default value.

```javascript
//bad
props: ['status']

props: {
  status: String
}

//good
props: {
  status: {
    type: String,
    'default': ''
  }
}

// Even better!
props: {
  status: {
    type: String,
    'default': '',
    required: true,
    validator: value => {
      return [ 'syncing', 'synced', 'error' ].includes(value);
    }
  }
}
```

While Vue.js supports passing complex JavaScript objects via these attributes, you should try to keep the component props as primitive as possible. Try to only use JavaScript primitives (strings, numbers, booleans) and functions. Avoid complex objects.

**Prop name casing**

**Prop names should always use camelCase during declaration, but kebab-case in templates and JSX.**

We’re simply following the conventions of each language. Within JavaScript, camelCase is more natural. Within HTML, kebab-case is.

```javascript
//bad
props: {
  'greeting-text': String
}
```
```html
<WelcomeMessage greetingText="hi"/>
```

```javascript
//good
props: {
  greetingText: String
}
```
```html
<WelcomeMessage greeting-text="hi"/>
```

**[⬆ back to top](#table-of-contents)**

### 4.3 `computed`

#### Keep them simple
*Complex computed properties should be split into as many simpler properties as possible.*

```javascript
//bad
computed: {
  price: function () {
    var basePrice = this.manufactureCost / (1 - this.profitMargin)
    return (
      basePrice -
      basePrice * (this.discountPercent || 0)
    )
  }
}

//good
computed: {
  basePrice: function () {
    return this.manufactureCost / (1 - this.profitMargin)
  },
  discount: function () {
    return this.basePrice * (this.discountPercent || 0)
  },
  finalPrice: function () {
    return this.basePrice - this.discount
  }
}
```

**[⬆ back to top](#table-of-contents)**

#### Keep them synchronous
> eslint: [`vue/no-async-in-computed-properties`](https://eslint.vuejs.org/rules/no-async-in-computed-properties.html#vue-no-async-in-computed-properties)

Computed properties should be synchronous. Asynchronous actions inside them may not work as expected and can lead to an unexpected behaviour, that's why you should avoid them. If you need async computed properties you might want to consider using additional plugin [vue-async-computed]

```javascript
//bad
computed: {
  foo1: async function () {
    return await someFunc()
  },
}

//good
computed: {
  foo1 () {
    var bar = 0
    try {
      bar = bar / this.a
    } catch (e) {
      return 0
    } finally {
      return bar
    }
  },
}
```

**[⬆ back to top](#table-of-contents)**

#### Keep them without side effects
> eslint: [`vue/no-side-effects-in-computed-properties`](https://eslint.vuejs.org/rules/no-side-effects-in-computed-properties.html#vue-no-side-effects-in-computed-properties)

It is considered a very bad practice to introduce side effects inside computed properties. It makes the code not predictable and hard to understand.

```javascript
//bad
computed: {
  fullName () {
    this.firstName = 'lorem' // <- side effect
    return `${this.firstName} ${this.lastName}`
  },
  reversedArray () {
    return this.array.reverse() // <- side effect - orginal array is being mutated
  }
}

//good
computed: {
  fullName () {
    return `${this.firstName} ${this.lastName}`
  },
  reversedArray () {
    return this.array.slice(0).reverse() // .slice makes a copy of the array, instead of mutating the orginal
  }
}
```

**[⬆ back to top](#table-of-contents)**

### Always return in a computed property
> eslint: [`vue/return-in-computed-property`](https://eslint.vuejs.org/rules/return-in-computed-property.html#vue-return-in-computed-property)

```javascript
//bad
computed: {
  baz () {
    if (this.baf) {
      return this.baf
    }
  }
}

//good
computed: {
  baz () {
    if (this.baf) {
      return this.baf
    }
    return false;
  }
}
```

**[⬆ back to top](#table-of-contents)**

## 5. `v-for` / `v-if`
<a name="vfor-vif"></a>

### 5.1 Always use `key` with `v-for`

`key` with `v-for` is **always** required on components, in order to maintain internal component state down the subtree. Even for elements though, it’s a good practice to maintain predictable behavior, such as object constancy in animations.

```html
//bad
<ul>
  <li v-for="todo in todos">
    {{ todo.text }}
  </li>
</ul>

//good
<ul>
  <li v-for="todo in todos" :key="todo.id" >
    {{ todo.text }}
  </li>
</ul>
```

### 5.2 Avoid using `v-if` with `v-for`

**Never use `v-if` on the same element as `v-for`.**

There are two common cases where this can be tempting:

* To filter items in a list (e.g. `v-for="user in users" v-if="user.isActive"`). In these cases, replace users with a new computed property that returns your filtered list (e.g. `activeUsers`).

* To avoid rendering a list if it should be hidden (e.g. `v-for="user in users" v-if="shouldShowUsers"`). In these cases, move the `v-if` to a container element (e.g. `ul`, `ol`).

```html
//bad
<ul>
  <li v-for="user in users" v-if="user.isActive" :key="user.id" >
    {{ user.name }}
  </li>
</ul>

-- or --

<ul>
  <li v-for="user in users" v-if="shouldShowUsers" :key="user.id" >
    {{ user.name }}
  </li>
</ul>

//good
<ul>
  <li v-for="user in activeUsers" :key="user.id" >
    {{ user.name }}
  </li>
</ul>

-- or --

<ul v-if="shouldShowUsers">
  <li v-for="user in users" :key="user.id">
    {{ user.name }}
  </li>
</ul>
```

**[⬆ back to top](#table-of-contents)**

## 6. Simple expressions in templates
<a name="simple-expressions-templates"></a>

**Component templates should only include simple expressions, with more complex expressions refactored into computed properties or methods.**

Complex expressions in your templates make them less declarative. We should strive to describe *what* should appear, not *how* we’re computing that value. Computed properties and methods also allow the code to be reused.

```html
//bad
<div>
  {{
    fullName.split(' ').map(function (word) {
      return word[0].toUpperCase() + word.slice(1)
    }).join(' ')
  }}
</div>


//good
<div>{{ normalizedFullName }}</div>
```
```javascript
// The complex expression has been moved to a computed property
computed: {
  normalizedFullName() {
    return this.fullName.split(' ').map(word => {
      return word[0].toUpperCase() + word.slice(1)
    }).join(' ');
  }
}
```

**[⬆ back to top](#table-of-contents)**

## 7. Quoted attribute values
<a name="quoted-attribute-values"></a>

> eslint: [`vue/html-quotes`](https://eslint.vuejs.org/rules/html-quotes.html#vue-html-quotes)

**Non-empty HTML attribute values should always be inside quotes (single or double, whichever is not used in JS).**

While attribute values without any spaces are not required to have quotes in HTML, this practice often leads to avoiding spaces, making attribute values less readable.

```html
//bad
<input type=text>

<AppSidebar :style={width:sidebarWidth+'px'}>

//good
<input type="text">

<AppSidebar :style="{ width: sidebarWidth + 'px' }">
```

**[⬆ back to top](#table-of-contents)**

## 7. Directive shorthands
<a name="directive-shorthands"></a>

**Directive shorthands (`:` for `v-bind:` and `@` for `v-on:`) should always be used.**

```html
//bad
<input
  v-bind:value="newTodoText"
  v-on:focus="newTodoFocus"
>

//good
<input
  :value="newTodoText"
  @focus="newTodoFocus"
>
```

**[⬆ back to top](#table-of-contents)**

## 8. Implicit parent-child communication
<a name="parent-child-communication"></a>

**Props and events should be preferred for parent-child component communication, instead of this.$parent or mutating props.**

An ideal Vue application is props down, events up. Sticking to this convention makes your components much easier to understand. However, there are edge cases where prop mutation or this.$parent can simplify two components that are already deeply coupled.

The problem is, there are also many simple cases where these patterns may offer convenience. Beware: do not be seduced into trading simplicity (being able to understand the flow of your state) for short-term convenience (writing less code).

[More info](https://vuejs.org/v2/style-guide/#Implicit-parent-child-communication-use-with-caution)

**[⬆ back to top](#table-of-contents)**

## 9. Non-flux state manangement
<a name="non-flux"></a>

[Vuex](https://github.com/vuejs/vuex) should be preferred for global state management, instead of `this.$root` or a global event bus.

Managing state on `this.$root` and/or using a [global event bus](https://vuejs.org/v2/guide/migration.html#dispatch-and-broadcast-replaced) can be convenient for very simple cases, but are not appropriate for most applications. Vuex offers not only a central place to manage state, but also tools for organizing, tracking, and debugging state changes.

[More info](https://vuejs.org/v2/style-guide/#Non-flux-state-management-use-with-caution)

**[⬆ back to top](#table-of-contents)**

## 10. Testing
<a name="testing"></a>

* Whichever testing framework you use, you should be writing tests!
* Strive to write many small pure functions, and minimize where mutations occur.
* Be cautious about stubs and mocks - they can make your tests more brittle.
* We primarily use mjest at Monosolutions.
* 100% test coverage is a good goal to strive for, even if it’s not always practical to reach it.
* Whenever you fix a bug, write a regression test. A bug fixed without a regression test is almost certainly going to break again in the future.

[Vue Test Utils](https://vue-test-utils.vuejs.org/) is the official unit testing utility library for Vue.js. It provides many helper methods to easily get started with testing with Vue.

**[⬆ back to top](#table-of-contents)**

## 10. ESLint
<a name="eslint"></a>

Linting for single file components in vue can be done using [ESLint](https://eslint.org/) and [eslint-plugin-vue](https://eslint.vuejs.org/), which is the official eslint plugin for Vue.js

* Example [`.eslintrc`](./linters/.eslintrc)
* Available rules: [Vue ESLint Rules](https://eslint.vuejs.org/rules/)

**[⬆ back to top](#table-of-contents)**

## 11. Resources
<a name="resources"></a>

### Websites

* [VueJS Documentation](https://vuejs.org/v2/guide/)
* [Vuex Documentation](https://vuex.vuejs.org/)

### Tools

* [Vue ESLint Plugin](https://eslint.vuejs.org)
* [Vue Test Utils](https://vue-test-utils.vuejs.org/)
* [Vue Jest Transformer](https://github.com/vuejs/vue-jest)
* [Webpack loader for Vue.js components](https://github.com/vuejs/vue-loader)

# };