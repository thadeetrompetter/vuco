# Vue.js

Make a Vue instance:

```html
<div id="some-element">
  {{ foo }} - {{ bar() }}
</div>
```
```javascript
new Vue({
  el: '#some-element',
  data: {
    foo: 'foo'
  },
  methods: {
    bar: function () {
      return this.foo;
    }
  }
})
```

All members of `data` and `methods` are available in the template without
prefix, as well as to each other.

## Directives

### `v-bind:attr="value"`

You cannot use curly brace interpolation for setting attribute values, you have
to use `v-bind`.

```javascript
new Vue({
  el: '#some-element',
  data: {
    foo: 'foo'
  }
})

```
```html
<a id="some-element" v-bind:href="foo"></a>
```

### `v-once`

Makes sure variables are interpolated only once within the scope of this node,
keeping the initial value of the binding in place. This directive does not
receive an argument.

### `v-html="htmlstring"`

By default, vue escapes anything it outputs. Use this binding if you want to
render html from a variable onto the page.

### `v-on:event="handlerfunc"`

Attach an event handler to an event. You can use any DOM standard event type.

#### The event object

The attached function receives an event object as first argument.

```javascript
new Vue({
  methods: {
    handlerFunc(event){
      // do something with the event object
    }
  }
});
```

#### Handler parameters

You pass a parameter where you call the directive. What you pass will become
the arguments to the function. You can add the event object by adding a special
reserved $-prefixed parameter, which translates to the event object.

```html
<button v-bind:click="func('foo', $event)"></button>
```

```javascript
new Vue({
  methods: {
    handlerFunc(param, event){
      // do something param and event
    }
  }
});
```

#### Event modifiers

Declare additional behavior for an event. Event modifiers can be chained.

E.g.: You want to listen for `mousemove` inside a div, but not when hovering a
specific child element of that div. Similar to calling `event.stopPropagation`

```html
<div v-on:mousemove="move">
  <span>i like to move it</span>
  <!-- Separate the modifier from the event with a period -->
  <div class="or-maybe-not" v-on:mousemove.stop>x: {{ x }} y: {{ y }}</div>
</div>
```

```javascript
new Vue({
  // ..truncated..
  data: {
    x: 0,
    y: 0
  },
  methods:{
    move(event){
      this.x = event.clientX;
      this.y = event.clientY;
    }
  }
});
```

##### Other modifiers

* **prevent** = `event.preventDefault`
* **[key]** = for keyboard events. will only execute the handler when pressing
given key(s). Can be chained.

[More modifiers](https://vuejs.org/v2/api/#v-on).

### `v-model`

Enables 2-way data binding. Typing in the field will update the value of the
`foo` property on the model.

```html
<input v-model="foo">
<span>{{ foo }}</span>
```

```javascript
new Vue({
  // ..truncated..
  data: {
    foo: 'bar'
  }
})
```

## Computed properties

Similar to methods, but declared as properties of `computed` in the view-model.
You invoke a computed property like you would interpolate a regular data
property. Computed track changes to data properties they depend on and as such
are more efficient than methods.

```html
<span>{{ bar }}</span>
<input v-model="foo" />
```

```javascript
// ..truncated..
data: {
  foo:''
},
computed: {
  bar: function () {
    return this.foo === 'world' ? 'hello' : 'wait for it...';
  }
}
```

## Watch

React to changes to a property. The property on the `watch` object
should correspond to a property on the `data` or `computed` object. Its
function value receives the changed value of the data property as an argument.
You should favor the use of computed properties over watch if you can. However,
if you are executing asynchronous code, then **watch** is the way to go.

```html
<span v-on:click="foo++">{{ value }}</span>
```

```javascript
new Vue({
  // ..truncated..
  data: {
    foo: 0,
    timeout: null,
    value: 'ready?'
  },
  watch: {
    foo: function () {
      var vm = this;
      if(this.timeout){
        clearTimeout(this.timeout); // debounce
      }
      this.timeout = setTimeout(function(){
        vm.value = vm.foo + ' go!';
      }, 2000);
    }
  }
});
```

## Directive shorthands

More concise ways to write directives.

For event handlers:

```html
<span v-on:click="someHandler">foo</span>
<!-- becomes -->
<span @click="someHandler">foo</span>
```

For attributes

```html
<a v-bind:href="http://example.com">foo</a>
<!-- becomes -->
<a :href="http://example.com">foo</a>
```

## CSS

To add classes to an element, use the `class` attribute binding and pass an
object with the class names to add being property names on the object. Its
property values' evaluation to booleans determines whether to show the
classnames or not. If you have a regular class property on the element, its
contents will be merged with the dynamically generated result.

```html
<div class="static-class" :class="computedClasses">some element</div>
<button @click="switchClasses">toggle classes</button>
```

```javascript
new Vue({
  // ..truncated..
  data: {
    foo: false,
    bar: true
  },
  methods: {
    switchClasses: function () {
      this.foo = !this.foo;
      this.bar = !this.bar;
    }
  },
  computed: {
    computedClasses: function () {
      return {
        'has-foo': this.foo,
        'has-bar': this.bar
      };
    }
  }
});
```

If you want to dynamically determine the classname(s) to add to the element,
you don't use an object, but instead pass a string or an array of class names.

If you decide to pass an array, you can decide to mix syntax, by including
objects (like in the above example) as elements of the array. Obviously, the
strings you specify, can be set dynamically.

```html
<!-- specify a string -->
<div class="static-class" :class="foo">some element</div>
<!-- an array of strings -->
<div class="static-class" :class="['foo', 'bar']">some element</div>
<!-- or a mixture of strings and class objects with conditionals -->
<div class="static-class" :class="['foo', { bar: false }]">some element</div>
```

### style

Using the `:style` directive, you can apply same object syntax you use to add
css classes to add inline styles, where the keys of the object are strings or
camelCase css properties with values.

```html
<div style="color: red" :style="{ backgroundColor: 'green' }">some element</div>
```

## Conditionals

### `v-if="expression"`

To conditionally show or hide an element and all of its child nodes. This will
completely remove the element from the dom if the expression evaluates to
false.

### `v-else`

Refers to the last `v-if` that came before it. Reversing the effect of the
condition for the element it is attached to.

When dealing with sibling elements you want to show/hide as a group, you can
wrap them in a `<template>` html tag, to which you will attach the directive.

```html
<template v-if="showFirst">
  <h1>some heading</h1>
  <p>some paragraph</p>
</template>
<template v-else>
  <h1>alternative heading</h1>
  <p>alternative paragraph</p>
</template>
<button @click="showFirst = !showFirst">alternate</button>
```

```javascript
new Vue({
  // ..truncated..
  data: {
    showFirst: true
  }
})
```

### `v-show="expression"`

Applies an inline style of `display: none` to an element.

## Lists

### `v-for`

To iterate over a collection. The element that the directive is attached to,
will be replicated. You will have access to the data for each iteration. The
first element will be the data for the current iteration and the select element
is the loop index. If you have sibling elements to render, wrap them in a
template tag.

You can also loop over the properties and values of an object. The order of
properties will not be guaranteed.

```html
<ul>
  <!-- loop over an array of strings -->
  <li v-for="(thing, index) in simpleThings">{{ index }}: {{ thing }}</li>
</ul>
<dl>
  <!-- loop over an array of objects -->
  <template v-for="thing in complexThings">
    <dt>id: {{ thing.id}}</dt>
    <dd>{{ thing.value }}</dd>
  </template>
</dl>
<ol>
  <!-- loop over the properties, values and indices of an object -->
  <li v-for="(value, key, index) in objectThings">
    <span>index: {{ index }}</span>,
    <span>key: {{ key }}</span>,
    <span>value: {{ value }}</span>
  </li>
</ol>
```

```javascript
new Vue({
  // ..truncated..
  data: {
    simpleThings: ['foo','bar','baz'],
    complexThings: [
      {id: 1000, value: 'foo'},
      {id: 1001, value: 'bar'},
      {id: 1002, value: 'baz'}
    ],
    objectThings: {
      foo: 'one',
      bar:'two',
      baz: 'three'
    }
  }
})
```

A special feature of `v-for` is its ability to loop over a range of numbers.

```html
<div v-for="number in 10">the number is {{ number }}</div>
```

[A note on vue's looping behavior and unique keys](https://vuejs.org/v2/guide/list.html#key).

## The Vue instance

It's possible to have multiple instances of `Vue` on the same page. These
instances can then also talk to each other. By saving the instances to a
variable, you have access to the members on either of the view models.
Properties, computed properties and methods are proxied to be available
directly on the instance.

```html
<section id="app-one">
  <h1>{{ title }}</h1>
</section>
<section id="app-two" @click="chanceAppOneTitle">
  <h1>{{ title }}</h1>
</section>
```

```javascript
const appOne = new Vue({
  el: '#app-one',
  data: {
    title: 'first app'
  }
});
const appTwo = new Vue({
  el: '#app-two',
  data: {
    title: 'second app'
  },
  methods: {
    chanceAppOneTitle() {
      // Direct access to app one's view-model.
      // Be aware that this does introduce very tight coupling...
      appOne.title = 'changed by app two';
    }
  }
});
```

Writing to the vue instance from the outside will not allow you to use the
property like you would use other data properties, because vue only sets up
proxying behavior for properties that you pass in the initial constructor call,
not for properties that you add later.

### `$`

Properties prefixed with `$` are properties on the vue instance that are OK
for you to use.

#### $el
Refers to the DOM element that your vue instance is bound to.

#### $data
Is a reference to the `data` property which was passed at init.

#### $refs
Additional hooks you can place on elements by adding a `ref` attribute with a
string value. This allows you to reference dom elements. `$refs` will be an
object with the refs you specified in templates as keys. E.g.:

`this.$refs.someElement`

You will not override binding present in the template this way. The template
that is stored by vue will always override change you made using refs.

#### $mount
To bind a Vue instance to an element after having constructed the instance.
Pass a selector string. This is useful if for whatever reason you want to defer
mounting the view-model to a DOM element. For instance, if the DOM to mount on
doesn't exist yet.

```html
<section id="app">
  <h1>{{ title }}</h1>
</section>
```
```javascript
const app = new Vue({
  data: {
    title: 'my app'
  }
});
app.$mount('#app');
```

### The `template` property
Pass an html string at construct time, which will become the DOM representation
of your view-model. You can append this to the DOM using `$mount`.

```html
<section id="app"></section>
```

```javascript
const app = new Vue({
  data: {
    title: 'my app'
  },
  template: `<h1>
              <span>{{ title }}</span>
             <h1>`
});
app.$mount('#app');

// alternatively:

app.mount();
document.getElementById('app').appendChild(app.$el);
```

### Components
Mount vue to a custom element, creating a re-usable component.

```html
<section id="app">
  <my-component></my-component>
</section>
```

```javascript
// Registers a component
Vue.component('my-component', {
  template: '<span>respect the component</span>'
});
new Vue({
  el:'#app'
})
```

### Vue instance life-cycle

#### `beforeCreate`
After calling vue constructor. Before attaching data and handlers to the
instance.
#### `created`
Instance has been created. Template is taken from the DOM or parsed from given
html string.
#### `beforeMount`
Done parsing the template in the virtual dom, interpolated all values.
#### `mounted`
elements mounted to actual DOM.
#### `beforeUpdate`
Data has changed, warranting a re-render, vue is about to re-render.
#### `updated`
New template has been attached to the DOM
#### `beforeDestroy`
Before instance is removed. To destroy a vue instance, call its `$destroy`
method.
#### `destroyed`
instance is removed.

## Components
A Vue instance using the `el` property to select an element to bind to, can
only be bound to a single (the first) element. Components allow you to re-use
logic and template markup, in the form of custom elements.

```javascript
Vue.component('my-component', {
  template: `<h1>{{ title }}</h1>`,
  // component data can not be a property, but has be a function.
  data(){
    return { title: 'this is a component' }
  }
});
new Vue({
  el: '#app'
})
```

### data
The reason that `data` has to be returned from a function, is because
objects are reference types. If the data in the component definition would be
an object, all instances of the component would be using the same data.

### Local components
`Vue.component` registers components globally and makes them available for use
by all vue instances. You can also register component locally for use by a
specific vue instance.

```html
<div id="app">
  <my-component>this text will be replaced</my-component>
</div>
<div id="another-app">
  <my-component>my-component does nothing here...</my-component>
</div>
```

```javascript
const myComponentConfig = {
  template: `<h1>{{ title }}</h1>`,
  data(){
    return { title: 'my local component' }
  }
};
// local component registration
new Vue({
  el: '#app',
  components: {
    'my-component': myComponentConfig
  }
});
// the other view-model does not know about my-component
new Vue({
  el: '#another-app'
});
```

### Vue files
Vue files contain template markup, script and styles. Project is scaffolded
with vue-cli's **webpack basic** template.

```html
<!-- MyComponent.vue -->
<template>
  <!-- if you have sibling elements, wrap them -->
  <div>
    <h1>welcome to my {{ variable }}</h1>
    <span>sibling element</span>
  </div>
</template>

<script>
  export default {
    data(){
      return {
        variable: 'template'
      }
    }
  }
</script>

<style>
body {
  background-color: yellow;
}
</style>
```

```javascript
// main.js
import Vue from 'vue';
import App from './App.vue';

// import component
import MyComponent from './MyComponent.vue';

// register component
Vue.component('my-component', MyComponent);

new Vue({
  el: '#app',
  render: h => h(App)
})

```

```html
<!-- use component in App.vue -->
<template>
  <my-component></my-component>
</template>
```

When the component is rendered in the DOM, it's not wrapped in a
`<my-component>` tag.

### Local sub-components
If a component wraps another component and there's no need to declare the
wrapped component globally, you can declare it local to the containing
component.

```html
<!-- MyComponent.vue -->
<template>
  <div>
    <h1>welcome to my {{ variable }}</h1>
    <!-- You can make as many instances of this as you want -->
    <my-other-component></my-other-component>
  </div>
</template>

<script>
  // local component imports are declared inside the importing component
  import MyOtherComponent from './MyOtherComponent.vue'
  export default {
    data(){
      return {
        variable: 'component'
      }
    },
    components: {
      'my-other-component': MyOtherComponent
    }
  }
</script>

<style>
body {
  background-color: yellow;
}
</style>

<!-- MyOtherComponent.vue -->
<template>
  <div>
    <h1>{{ interpolateMe }}</h1>
    <button @click="changeInterpolation">click here..</button>
  </div>
</template>

<script>
  export default {
    data(){
      return {
        interpolateMe: 'all night long'
      }
    },
    methods: {
      changeInterpolation(){
        this.interpolateMe = 'day after day';
      }
    }
  }
</script>
```

```javascript
// same as javascript in 'Vue files' section
```

### Case in custom tags
Because your vue files will be compiled to javascript, it's OK to use case
sensitive tag names. Even if you use dash-case in the DOM, and camelCase in
component registration.
