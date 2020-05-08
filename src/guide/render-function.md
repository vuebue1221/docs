# Render Functions & JSX

> This section is incomplete which means some information could be missing or outdated.

## Basics

Vue recommends using templates to build your HTML in the vast majority of cases. There are situations however, where you really need the full programmatic power of JavaScript. That's where you can use the **render function**, a closer-to-the-compiler alternative to templates.

Let's dive into a simple example where a `render` function would be practical. Say you want to generate anchored headings:

```html
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```

For the HTML above, you decide you want this component interface:

```vue-html
<anchored-heading :level="1">Hello world!</anchored-heading>
```

When you get started with a component that only generates a heading based on the `level` prop, you quickly arrive at this:

```js
const app = Vue.createApp({})

app.component('anchored-heading', {
  template: `
    <h1 v-if="level === 1">
      <slot></slot>
    </h1>
    <h2 v-else-if="level === 2">
      <slot></slot>
    </h2>
    <h3 v-else-if="level === 3">
      <slot></slot>
    </h3>
    <h4 v-else-if="level === 4">
      <slot></slot>
    </h4>
    <h5 v-else-if="level === 5">
      <slot></slot>
    </h5>
    <h6 v-else-if="level === 6">
      <slot></slot>
    </h6>
  `,
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

That template doesn't feel great. It's not only verbose, but we're duplicating `<slot></slot>` for every heading level and will have to do the same when we add the anchor element.

While templates work great for most components, it's clear that this isn't one of them. So let's try rewriting it with a `render` function:

```js
const app = Vue.createApp({})

app.component('anchored-heading', {
  render() {
    const { h } = Vue

    return h(
      'h' + this.level, // tag name
      this.$slots.default() // array of children
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

<!-- TODO: Reword following section because "familiarity with instance properties" is misleading as `setup()` function does not have direct access to the instance.  -->

Much simpler! The code is shorter, but also requires greater familiarity with Vue instance properties. In this case, you have to know that when you pass children without a `v-slot` directive into a component, like the `Hello world!` inside of `anchored-heading`, those children are stored on the component instance at `$slots.default`. If you haven't already, **it's recommended to read through the [instance properties API](#TODO-add-link) before diving into render functions.**

## Nodes, Trees, and the Virtual DOM

Before we dive into render functions, it’s important to know a little about how browsers work. Take this HTML for example:

```html
<div>
  <h1>My title</h1>
  Some text content
  <!-- TODO: Add tagline  -->
</div>
```

When a browser reads this code, it builds a [tree of "DOM nodes"](https://javascript.info/dom-nodes) to help it keep track of everything, just as you might build a family tree to keep track of your extended family.

The tree of DOM nodes for the HTML above looks like this:

![DOM Tree Visualization](/images/dom-tree.png)

Every element is a node. Every piece of text is a node. Even comments are nodes! A node is just a piece of the page. And as in a family tree, each node can have children (i.e. each piece can contain other pieces).

Updating all these nodes efficiently can be difficult, but thankfully, you never have to do it manually. Instead, you tell Vue what HTML you want on the page, in a template:

```html
<h1>{{ blogTitle }}</h1>
```

Or a render function:

```js
render() {
  return Vue.h('h1', this.blogTitle)
}
```

And in both cases, Vue automatically keeps the page updated, even when `blogTitle` changes.

### The Virtual DOM

Vue accomplishes this by building a **virtual DOM** to keep track of the changes it needs to make to the real DOM. Taking a closer look at this line:

```js
return Vue.h('h1', this.blogTitle)
```

What is `h()` actually returning? It's not _exactly_ a real DOM element. It could perhaps more accurately be named `createNodeDescription`, as it contains information describing to Vue what kind of node it should render on the page, including descriptions of any child nodes. We call this node description a "virtual node", usually abbreviated to **VNode**. "Virtual DOM" is what we call the entire tree of VNodes, built by a tree of Vue components.

## `h()` Arguments

The next thing you'll have to become familiar with is how to use template features in the `h()` function. Here are the arguments that `h()` accepts:

```js
// @returns {VNode}
h(
  // {String | Object | Function}
  // An HTML tag name, component options, or async
  // function resolving to one of these. Required.
  'div',

  // {Object}
  // An object corresponding to the attributes
  // you would use in a template. Optional.
  {
    // (see details in the next section below)
  },

  // {String | Array | Object}
  // Children VNodes, built using `h()`,
  // or using strings to get 'text VNodes' or
  // an object with slots. Optional.
  [
    'Some text comes first.',
    h('h1', 'A headline'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ]
)
```

### The Data Object In-Depth

> TODO: This section is entirely new and would be done in a separate PR.

### Complete Example

With this knowledge, we can now finish the component we started:

```js
const app = Vue.createApp({})

function getChildrenTextContent(children) {
  return children
    .map(node => {
      // TODO: Check new vnode API.
      return node.children ? getChildrenTextContent(node.children) : node.text
    })
    .join('')
}

app.component('anchored-heading', {
  render() {
    // create kebab-case id
    const headingId = getChildrenTextContent(this.$slots.default())
      .toLowerCase()
      .replace(/\W+/g, '-') // replace non-word characters with dash
      .replace(/(^-|-$)/g, '') // remove leading and trailing dashes

    return Vue.h('h' + this.level, [
      Vue.h(
        'a',
        {
          name: headingId,
          href: '#' + headingId
        },
        this.$slots.default()
      )
    ])
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

### Constraints

#### VNodes Must Be Unique

All VNodes in the component tree must be unique. That means the following render function is invalid:

```js
render() {
  const myParagraphVNode = Vue.h('p', 'hi')
  return Vue.h('div', [
    // Yikes - duplicate VNodes!
    myParagraphVNode, myParagraphVNode
  ])
}
```

If you really want to duplicate the same element/component many times, you can do so with a factory function. For example, the following render function is a perfectly valid way of rendering 20 identical paragraphs:

```js
render() {
  return Vue.h('div',
    Array.apply(null, { length: 20 }).map(() => {
      return Vue.h('p', 'hi')
    })
  )
}
```

## Replacing Template Features with Plain JavaScript

### `v-if` and `v-for`

Wherever something can be easily accomplished in plain JavaScript, Vue render functions do not provide a proprietary alternative. For example, in a template using `v-if` and `v-for`:

```html
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```

This could be rewritten with JavaScript's `if`/`else` and `map` in a render function:

```js
props: ['items'],
render() {
  if (this.items.length) {
    return Vue.h('ul', this.items.map((item) => {
      return Vue.h('li', item.name)
    }))
  } else {
    return Vue.h('p', 'No items found.')
  }
}
```

### `v-model`

~~There is no direct `v-model` counterpart in render functions - you will have to implement the logic yourself:~~

TODO: This section requires update.

```js
props: ['modalValue'],
render() {
  return Vue.withDirectives(
    Vue.h('input', {
      'onUpdate:modelValue': value => this.$emit('onUpdate:modelValue', value)
    }),
    [Vue.vModelDynamic, this.modelValue]
  )
}
```

~~This is the cost of going lower-level, but it also gives you much more control over the interaction details compared to `v-model`.~~

### Event & Key Modifiers

> TODO: This section requires an update.

~~For the `.passive`, `.capture` and `.once` event modifiers, Vue offers prefixes that can be used with `on`:~~

For example:

```javascript
on: {
  '!click': this.doThisInCapturingMode,
  '~keyup': this.doThisOnce,
  '~!mouseover': this.doThisOnceInCapturingMode
}
```

For all other event and key modifiers, no proprietary prefix is necessary, because you can use event methods in the handler:

| Modifier(s)                                           | Equivalent in Handler                                                                                                |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `.stop`                                               | `event.stopPropagation()`                                                                                            |
| `.prevent`                                            | `event.preventDefault()`                                                                                             |
| `.self`                                               | `if (event.target !== event.currentTarget) return`                                                                   |
| Keys:<br>`.enter`, `.13`                              | `if (event.keyCode !== 13) return` (change `13` to [another key code](http://keycode.info/) for other key modifiers) |
| Modifiers Keys:<br>`.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (change `ctrlKey` to `altKey`, `shiftKey`, or `metaKey`, respectively)                  |

Here's an example with all of these modifiers used together:

```javascript
onKeyup: function (event) {
  // Abort if the element emitting the event is not
  // the element the event is bound to
  if (event.target !== event.currentTarget) return
  // Abort if the key that went up is not the enter
  // key (13) and the shift key was not held down
  // at the same time
  if (!event.shiftKey || event.keyCode !== 13) return
  // Stop event propagation
  event.stopPropagation()
  // Prevent the default keyup handler for this element
  event.preventDefault()
  // ...
}
```

### Slots

You can access static slot contents as Arrays of VNodes from [`this.$slots`](../api/#vm-slots):

```js
render() {
  // `<div><slot></slot></div>`
  return Vue.h('div', null, this.$slots.default())
}
```

```js
props: ['message'],
render() {
  // `<div><slot :text="message"></slot></div>`
  return Vue.h('div', null, this.$slots.default({
    text: this.message
  }))
}
```

To pass slots to a child component using render functions:

```js
render() {
  // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
  return Vue.('div', [
    Vue.('child', null, {
      // pass `scopedSlots` in the data object
      // in the form of { name: props => VNode | Array<VNode> }
      default: (props) => Vue.h('span', props.text)
    })
  ])
}
```

## JSX

If you're writing a lot of `render` functions, it might feel painful to write something like this:

```js
Vue.h(
  'anchored-heading',
  {
    level: 1
  },
  [Vue.h('span', 'Hello'), ' world!']
)
```

Especially when the template version is so simple in comparison:

```vue-html
<anchored-heading :level="1"> <span>Hello</span> world! </anchored-heading>
```

That's why there's a [Babel plugin](https://github.com/vuejs/jsx) to use JSX with Vue, getting us back to a syntax that's closer to templates:

```jsx
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render() {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```

For more on how JSX maps to JavaScript, see the [usage docs](https://github.com/vuejs/jsx#installation).

## Template Compilation

You may be interested to know that Vue's templates actually compile to render functions. This is an implementation detail you usually don't need to know about, but if you'd like to see how specific template features are compiled, you may find it interesting. Below is a little demo using `Vue.compile` to live-compile a template string:

<iframe src="http://vue-next-template-explorer.netlify.com/" width="100%" height="420"></iframe>
