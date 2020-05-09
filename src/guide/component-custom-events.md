# Custom Events

> This page assumes you've already read the [Components Basics](component-basics.md). Read that first if you are new to components.

## Event Names

Unlike components and props, event names don't provide any automatic case transformation. Instead, the name of an emitted event must exactly match the name used to listen to that event. For example, if emitting a camelCased event name:

```js
this.$emit('myEvent')
```

Listening to the kebab-cased version will have no effect:

```html
<!-- Won't work -->
<my-component v-on:my-event="doSomething"></my-component>
```

Since event names will never be used as variable or property names in JavaScript, there is no reason to use camelCase or PascalCase. Additionally, `v-on` event listeners inside DOM templates will be automatically transformed to lowercase (due to HTML's case-insensitivity), so `v-on:myEvent` would become `v-on:myevent` -- making `myEvent` impossible to listen to.

For these reasons, we recommend you **always use kebab-case for event names**.

## Defining Custom Events

Emitted events can be defined on the component via the `emits` option.

```js
app.component('custom-form', {
  emits: ['in-focus', 'submit']
})
```

In the event a native event (e.g., `click`) is defined in the `emits` option, it will be overwritten by the event in the component instead of being treated as a native listener.

::: tip
It is recommended to define all emitted events in order to better document how a component should work.
:::

### Validate Emitted Events

Similar to prop type validation, an emitted event can be validated if it is defined with the Object syntax instead of the Array syntax.

To add validation, the event is assigned a function that receives the arguments passed to the `$emit` call and returns a boolean to indicate whether the event is valid or not.

```js
app.component('custom-form', {
  emits: {
    // No validation
    click: null,

    // Validate submit event
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false
      }
    }
  },
  methods: {
    submitForm() {
      this.$emit('submit', { email, password })
    }
  }
})
```

## `v-model` arguments

By default, `v-model` on a component uses `modelValue` as the prop and `update:modelValue` as the event. We can modify these names passing an argument to `v-model`:

```html
<my-component v-model:foo="bar"></my-component>
```

In this case, child component will expect a `foo` prop and emits `update:foo` event to sync:

```js
const app = Vue.createApp({})

app.component('my-component', {
  props: {
    foo: String
  },
  template: `
    <input 
      type="text"
      v-bind:value="foo"
      v-on:input="$emit('update:foo', $event.target.value)">
  `
})
```

Note that this enables multiple v-model bindings on the same component, each syncing a different prop, without the need for extra options in the component:

```html
<my-component v-model:foo="bar" v-model:name="userName"></my-component>
```

## Handling `v-model` modifiers

In 2.x, we have hard-coded support for modifiers like `.trim` on component `v-model`. However, it would be more useful if the component can support custom modifiers. In 3.x, modifiers added to a component `v-model` will be provided to the component via the modelModifiers prop:

```html
<my-component v-model.capitalize="bar"></my-component>
```

```js
app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  template: `
    <input type="text" 
      v-bind:value="modelValue"
      v-on:input="$emit('update:modelValue', $event.target.value)">
  `,
  created() {
    console.log(this.modelModifiers) // { capitalize: true }
  }
})
```

We can check `modelModifiers` object keys and write a handler to change the emitted value. In the code below we will capitalize the string:

```html
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      myText: ''
    }
  }
})

app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  },
  template: `<input
    type="text"
    v-bind:value="modelValue"
    v-on:input="emitValue">`
})

app.mount('#app')
```

For `v-model` with arguments, the generated prop name will be `arg + "Modifiers"`:

```html
<my-component v-model:foo.capitalize="bar"></my-component>
```

```js
app.component('my-component', {
  props: ['foo', 'fooModifiers'],
  template: `
    <input type="text" 
      v-bind:value="foo"
      v-on:input="$emit('update:foo', $event.target.value)">
  `,
  created() {
    console.log(this.fooModifiers) // { capitalize: true }
  }
})
```
