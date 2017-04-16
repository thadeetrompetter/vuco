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
will be replicated. You have access to the data for each iteration. The
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
    changeAppOneTitle() {
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
// javascript is the same as javascript in 'Vue files' section
```

#### Case in custom tags
Because your vue files will be compiled to javascript, it's OK to use case
sensitive tag names. Even if you use dash-case in the DOM, and camelCase in
component registration. Vue figures it out so you can use compliant tag names
even though the component is registered by a camelCased name.

### Component style scoping
By default, styles you define in the `<style>` section of a component, will be
applied globally.
If you want your styles to only be applied to the component they are defined
in, add the `scoped` attribute.

```html
<!-- truncated -->
<style scoped>
div {
  background-color: green;
}
</style>
```

`Vue` makes this work by automagically adding a data-attribute to the element,
as well as a style tag in the head of the document referencing that attribute
with an attribute selector, thereby scoping the scoped css to the element.

## Component communication

### Parent => child
Using `props`, you pass data from the parent component to a child component.
To inform a child component that it will receive `props`, add this property to
the child component's view-model. Its value is an array of strings: The
property names you should be able to set. If you're using the property
interpolated in your template, it should match with a string in the array of
properties.

```html
<!-- component passing a property -->
<template>
  <receiving-component some-property="{{ someValue }}"></receiving-component>
</template>
<!-- component receiving the property -->
<template>
  <div>
    Check out my props: {{ some-property }}
  </div>
</template>
<script>
export default {
  props: ['some-property']
}
</script>
```

Using properties like this will pass by value. This means that you will not
have a dynamic updates of the value passed in to a component.

To have the passed property value behave dynamically, use `v-bind`:

```html
<template>
  <receiving-component v-bind:some-property="{{ someValue }}"></receiving-component>
  <!-- or -->
  <receiving-component :some-property="{{ someValue }}"></receiving-component>
</template>
```

Usage of props is not limited to template, you can also use props in methods.

```javascript
export default {
  methods: {
    replaceSomeValue(){
      return `replaced: ${this.someValue}`;
    }
  }
}
```

### Prop validation
To validate the properties being passed into a component, change the `props`
property to an object. You can add a single constructor of which type the
property is allowed to be, or you can add multiple constructors as an array.

Vue will error out if you pass the wrong type of value for a property.

```javascript
// ..truncated
props: {
  someValue: [ String, Object ]
}
```

You can take this a step further, by adding passing object instead of an array
of constructors.

```javascript
// ..truncated
props: {
  someValue: {
    type: String,
    required: true,
    // or
    default: 'my default value'
  }
}
```

If you intend to pass an object (or another reference type) as the default
value for a prop, make sure that it's returned from a function, for the same
reason you return it from a function when specifying component data.
Primitive can be returned as-is.

### Child => parent
To communicate changes from child to parent component, custom events are used.
Vue provides the `$emit` method for this, which receives an event name and,
optionally, some event data.

```html
<!-- child component -->
<script>
  export default {
    methods: {
      this.$emit('myCustomEvent', 'someEventData');
    }
  }
</script>
<!-- parent component -->
<template>
  <child-component @myCustomEvent="handleCustomEvent($event)">
</template>
<script>
  export default {
    methods: {
      handleCustomEvent(eventData){
        // you can now do stuff with the received event data.
      }
    }
  }
</script>
```

An alternative to using an event to communicate from child to parent, is to
have the parent component pass a callback function to the child component,
which is executed in the child component, but manipulates data in the parent.

Here, The `setName` method is defined on the parent, but executed as a click
handler inside the child component.

```html
<!-- parent component -->
<template>
  <div>
    <my-component :setName="setName"></my-component>
    <h1>{{ myName }}</h1>
  </div>
</template>
<script>
import MyComponent from './MyComponent.vue';
export default {
  data() {
    return {
      myName: 'foo'
    }
  },
  components: {
    myComponent: MyComponent
  },
  methods: {
    setName(){
      this.myName = 'bar';
    }
  }
}
</script>

<!-- child component -->
<template>
  <div>
    <button @click="setName">set name</button>
  </div>
</template>

<script>
  export default {
    props: {
      setName: {
        type: Function
      }
    }
  }
</script>
```

### Communication between sibling components
Sibling components should not communicate directly. Instead, the should
communicate through an event bus. This event bus can be the parent, where they
share a reference to a variable. Better even is to create a completely
separated event bus. This event bus is a Vue instance. Use that instance as an
event bus only, or attach other centralized data/behavior to it.

```javascript
// in main.js, before app bootstrap, export the event bus.
export const eventBus = new Vue();

// in a component
import { eventBus } from './main';
// in an event handler function in that component..
eventBus.$emit('myEvent', 'myEventData');

// in a sibling component
import { eventBus } from './main';
// attach a listener to the event bus during the component 'created' lifecycle
// hook
created(){
  eventBus.$on('myEvent', eventData => {
    // do something with event data
  });
}
```

### Slots
Allow you to transclude markup. Add a `<slot></slot>` where you want to output
the transcluded markup. The markup that is passed, receives its css from what
is declared in the component where it's rendered.

On the other hand, data interpolation happens based on the data that is
available in the parent component.

```html
<!-- parent component -->
<template>
  <my-component>
    <!-- interpolation happens in the parent component -->
    <h1>{{ heading }}</h1>
    <p>some text</p>
  </my-component>
</template>
<script>
  export default {
    data(){
      return {
        heading: 'some heading'
      }
    }
  }
</script>

<!-- child component -->
<template>
  <div>
    <slot></slot>
  </div>
</template>
<style>
  /* transcluded markup will receive styling from this declaration */
  h1 {
    background-color: yellow;
  }
</style>
```

#### Multiple (named) slots
By adding a `name` attribute to a slot, you can add multiple transclusion
points to your markup.

```html
<!-- In parent -->
<child-component>
  <h1 slot="title">Title</h1>
  <p slot="content">Lorem ipsum...</p>
</child-component>
<!-- In child -->
<div>
  <header>
    <slot name="title"></slot> <!-- Title -->
  </header>
  <main>
    <slot name="content"></slot> <!-- Lorem ipsum... -->
  </main>
</div>
```

#### Default slot
Not adding a name attribute to a single slot will make that particular slot the
default location to place content. Any content enclosed in the component's
tags that don't have a `slot` attribute defined, will be placed in the default
slot.

#### Placeholder content
Any `textContent` enclosed by a slot tag, is its default value. If you don't
specify content for that slot, it will be shown.

### Dynamic components
`<component></component>`
Is a reserved tagname, which acts as a slot for a **dynamic component**.
With the `:is` binding, you specify the name of the component that should be
loaded in its place.

```html
<!-- App.vue -->
<template>
  <div>
    <h1>dynamic components</h1>
    <button @click="selectedComponent = 'foo-component'">foo</button>
    <button @click="selectedComponent = 'bar-component'">bar</button>
    <component :is="selectedComponent"></component>
  </div>
</template>
<script>
  import MyComponent from './MyComponent.vue';
  import Foo from './FooComponent.vue';
  import Bar from './BarComponent.vue';

  export default {
    data(){
      return {
        selectedComponent: 'my-component'
      }
    },
    components: {
      myComponent: MyComponent,
      fooComponent: Foo,
      barComponent: Bar,

    }
  }
</script>

<!-- also loaded: -->
<!-- MyComponent.vue -->
<!-- FooComponent.vue -->
<!-- BarComponent.vue -->
```

#### State
By default, Switching between dynamic components will destroy and recreate
instances, so any state will be lost.

You can override this default behavior by wrapping the `component` in
`<keep-alive></keep-alive>` tags. This will make sure stored `data` survives.

#### Lifecycle hooks

`activated` and `deactivated` are hooks that relate specifically to dynamic
components. Whenever a dynamic component loads and becomes active, its
**activated** hook will fire. When un-loading this component, you'll see its
**deactivated** hook fire.

## Forms
`v-model` is used for 2-way data binding between your VM data and form inputs.
You can group related for data properties by nesting them inside an object.

You pre-populate form fields by filling in their values in the data object in
the VM.

```html
<template>
  <div>
    <form>
      <input type="text" v-model="userData.name">
      <input type="text" v-model="userData.age">
      <input type="text" v-model="userData.password">
    </form>
    <section>
      <p>name: { userData.name }}</p>
      <p>name: { userData.age }}</p>
      <p>name: { userData.password }}</p>
    </section>
  </div>
</template>
<script>
  export default {
    data(){
      userData: {
        name: 'The User',
        age: '33',
        password: 'secret'
      }
    }
  }
</script>
```

### Input modifiers

`v-model` takes optional modifiers to change the way it binds to the underlying
data.

* `v-model.lazy` Updates the data on **change** instead of on **input**.
* `v-model.trim` Trims excess whitespace from the value.
* `v-model.number` Converts value to a number. This only works if the value can
be converted to a number, else it will fall back to string.

### Working with textarea
With textareas, you have to use `v-model` to set a default value. Adding
text between the textarea's opening and closing tags, will not work.

To preserve line breaks in a textarea's value that is bound, set a regular
style attribute `style="white-space: pre"`.

### Working with checkboxes

To add the values of multiple checked checkboxes to an array, declare a data
property as an array and bind it to checkboxes. If you want to pre-select
checkboxes, add their values to the bound array up front.

```html
<!-- truncated -->
<input type="checkbox" value="apples" v-model="fruits"> apples
<input type="checkbox" value="oranges" v-model="fruits"> oranges
<!-- trucated -->
<script>
// truncated
data(){
  return {
    myName: 0,
    fruits: []
  }
}
// truncated
</script>
```

### The select element
Loop to create the options for a select. To mark an option as selected,
pre-populate the variable you bind with `v-model`, or add the `:selected`
property to the options. `v-model` will override `:selected`.

```html
<template>
  <div>
    <form>
      <select id="fruit" v-model="selectedFruit">
        <option v-for="fruit in fruits" :value="fruit">{{ fruit }}</option>
      </select>
      <output for="fruit">{{ selectedFruit }}</output>
    </form>
  </div>
</template>
<script>
  export default {
    data(){
      return {
        selectedFruit: 'oranges',
        fruits: [
          'apples',
          'oranges',
          'bananas'
        ]
      }
    }
  }
</script>
```

### Custom controls
To use `v-model` on a custom element, you have to implement its interface.
Under the hood, `v-model` just adds a `:value` binding to an element for
setting a value on the control, and also listens for events being emitted for
getting its value (`input` by default).

```html
<!-- in App.vue -->
<template>
  <div>
    <form>
      <my-control v-model="myControlState"></my-control>
      <output>my-control is {{ myControlState ? 'on' : 'off' }}</output>
    </form>
  </div>
</template>
<script>
  import MyControl from './MyControl.vue';
  export default {
    components: {
      myControl: MyControl
    },
    data(){
      return {
        myControlState: true
      }
    }
  }
</script>

<!-- in MyControl.vue -->
<template>
  <div>
    <form>
      <div class="control">
        <span @click="toggle(true)"><span v-if="isOn">✔</span> on</span>
        <span @click="toggle(false)"><span v-if="!isOn">✔</span> off</span>
      </div>
    </form>
  </div>
</template>
<script>
  export default {
    data(){
      return {
        isOn: this.value
      }
    },
    methods: {
      toggle(value){
        this.isOn = value;
        this.$emit('input', this.isOn)
      }
    },
    props: {
      value: Boolean
    }
  }
</script>
```

## Directives
To create a custom directive, you have to register it. This can be done
globally or locally.

```javascript
// global
Vue.directive('my-directive', { /* lifecycle methods */ })
// local to a component
export default {
  directives: {
    myDirective: {
      bind(el, binding, vNode) {
        // code here
      }
    }
  }
}
// directive is used `v-my-directive:argument.modifierN="value"`
```
You can pass a javascript object as value.

### Lifecycle
To add behavior to a directive, you can take advantage of 5 available lifecycle
hooks.

0. `bind(el, binding, vnode)` The directive is attached.
0. `inserted(el, binding, vnode)` Inserted in parent node.
0. `updated(el, binding, vnode, oldVnode)` Component is updated, but not its
children.
0. `componentUpdated(el, binding, vnode)` Component and children have been
updated.
0. `unbind(el, binding, vnode)` Directive is removed.

Values you pass to the directive are available at `binding.value`

```html
<my-component v-my-directive="'foobar'"></my-component>
```

### Arguments
To pass an argument to a custom directive, separate it from the directive name
with a colon. Arguments are always interpreted as strings and can't pass
more than one argument. The argument is available at `binding.arg`.

```html
<my-component v-my-directive:some-arg></my-component>
```

### Modifiers
Modifiers are separated from the directive name with a dot. They are
interpreted as boolean values, and you can have multiple. Modifiers become
properties of `binding.modifiers`.

## Filters
Modify view-model data for presentation. Declare filter globally or locally.
Filters can be chained.

```javascript
// global filter
Vue.filter('my-filter', value => {
  // return modified value
});
// local filter
filters: {
  myFilter (value) {
    // return modified value
  }
}
```

For performance reasons, you should prefer using a computed property over using
a filter.

### Usage
```html
<template>
  {{ myValue | filter-one | anotherFilter }}
</template>
```
## Mixins
To share data and behavior between multiple components, without the need to
duplicate code. A mixin is a plain javascript object. Its properties will be
merged with component properties where the mixin is applied. Component
properties will have priority over mixin properties. If, however, your are
adding lifecycle hooks by means of mixin, the lifeycle hooks will co-exist with
hooks already present on the component. The component always gets the
opportunity to override what a mixin adds.

```javascript
export default const myMixin {
  data() {
    return {
      someProperty: 'override me',
      anotherProperty: 'i will survive'
    }
  }
}
// in a component:
export default {
  mixins: [ myMixin ],
  data(){
    return {
      someProperty: 'someProperty the component wins'
      // anotherProperty will be added to the component data
    }
  }
}
```

## HTTP
Extend vue with **vue-resource** to allow it to conveniently make HTTP
requests.

```javascript
import Vue from 'vue';
import VueResource from 'vue-resource';

Vue.use(VueResource);
```

This will make the `$http` property available on all Vue instances. Outside of
vue instances, vue-resource is available as `Vue.http`.

`$http` provides methods to perform HTTP requests. It's similar to
`window.fetch`.

### POST
```javascript
this.$http.post('http://some-endpoint-url', this.myDataProperty)
  .then(console.log.bind(console))
  .catch(console.error.bind(console))
```

### GET
```javascript
this.$http.get('http://some-endpoint-url')
  .then(response => response.json())
  .then(data => {
    this.myValues = Object.values(data); // `this` is the Vue instance.
  })
  .catch(console.error.bind(console))
```

### Vue-resource options
To set global options for Vue resource, use `Vue.http.options`.

```javascript
// e.g.: to set a root url
Vue.http.options.root = 'https://my-global-endpoint';
```

Check out the [available options](https://github.com/pagekit/vue-resource/blob/develop/docs/config.md)

### HTTP interceptors (middleware)
Interceptors are stored as an array of functions, and are executed in order.
Interceptor functions receive a request object and a callback. If you want to
also manipulate the server response, you pass a callback to `next`. Note that
the 'response' callback can only do synchronous processing, because it does not
receive a `next` argument of its own.

```javascript
// Add an HTTP interceptor
Vue.http.interceptors.push((request, next) => {
  // sync/async do something with the request...
  next(response => {
    // synchronously do something with the response
  });
})
```

### `$resource`
An abstraction to `$http`, that maps a local VM property to a remote resource.

[resource manual](https://github.com/pagekit/vue-resource/blob/develop/docs/resource.md)

```javascript
// in a VM
export default {
  data(){
    return {
      resource: {},
      user: {
        username: '',
        email: ''
      }
    }
  },
  methods: {
    submitForm(){
      // first argument represents query string parameters
      this.resource.save({}, this.user);
      // this will post the current user data to the endpoint defined below.
    }
  },
  created(){
    this.resource = this.$resource('http://some-endpoint');
  }
}
```

#### Custom Actions
Extend resources with your own custom functionality.

```javascript
const customBehavior = {
  // this property will become a method on your resource.
  alternativePost: { method: 'POST', url: 'alternative.json' }
};
this.resource = this.$resource('data.json', {}, customBehavior);

// in a handler method:
this.resource.alternativePost({ /* data */ })
```

### Template URLs
Templating according to the standard described at
[uri.js](https://medialize.github.io/URI.js/uri-template.html).

```javascript
this.resource = this.$resource('{key}.json', {}, {
  getData: { method: 'GET' }
});

// pass an object to the custom handler to do some templating
this.resource.getData({ key: 'will-be-added-to-url' })
```
## Animation
Animate elements with the `<transition name="your-transition"></transition>`
tag. Vue will read css to find out how long an animation should take, and
attach css classes to the element wrapped in the `transition` tag.

Vue associates a transition with value of the `name` property set to
`your-transition` with the following css classes:

```css
.your-transition-enter {}
.your-transition-enter-active {}
.your-transition-leave {}
.your-transition-leave-active {}
```

You can use these styling hooks to respond to the element changing visibility.
You can use `animation` or `transition` to animate. If you want to coerce Vue
to look at the duration for either of these animation types, specify a `type`
property on the `transition` element.

### `appear`
This attribute makes the element transition in on page load.

### Custom animation classnames
Use `enter[-active]-class` and `leave[-active]-class` to override the default
animation classnames.

### Transitions between elements
Adding multiple elements to the same transition tag is possible, but this
requires you to add unique `key` properties to both elements, to make Vue
understand how to differentiate between them.

## Routing

```sh
npm install vue-router
```

```javascript
// in main.js
import Vue from 'vue';
import VueRouter from 'vue-router';
import App from './App.vue';
import User from './User.vue';
import Home from './Home.vue';

// load router plugin
Vue.use(VueRouter);

// It's a good idea to declare your routes at root level
const routes = [
  { path: '', component: Home },
  { path: '/user', component: User }
];

// instantiate a router with the defined routes
const router = new VueRouter({ routes });

new Vue({
  el: '#app',
  router,
  render: h => h(App)
})
```

```html
<!-- App.vue -->
<template>
  <div>
    <h1>Routing</h1>
    <!-- Router will mount components here -->
    <router-view></router-view>
  </div>
</template>

<!-- User.vue -->
<template>
  <h1>User route</h1>
</template>

<!-- Home.vue -->
<template>
  <h1>Home route</h1>
</template>
```

### `<router-link>`

a replacement for the regular anchor tag, which ensures that no http request
goes out to the server for clicking on a link.

```html
<router-link to="/absolute-path"></router-link>
<router-link to="relative-path"></router-link>
```

Inserting a `router-link` will translate to an anchor tag on the page with its
default behavior prevented.

#### Alternative linkage
To use a different element from the default anchor that `router-link` places on
the page, you can attach the link behavior to an alternative element with the
`tag` attribute. This is handy if you want to display the active state on a
list item in a list, instead of on the anchor nested inside.

0. The `active-class` attribute allows overriding the default class
`router-link-active`
0. `exact` will make sure the uri path only triggers an active link when it
exactly matches the value of the `to` attribute.

```html
<ul>
  <router-link to="/" tag="li" active-class="active" exact><a>main</a></router-link>
  <router-link to="/other-route" tag="li" active-class="active"><a>other</a></router-link>
</ul>
```

### The `$router` instance
You control the router programmatically via `this.$router`.

0. `this.$router.push()` to navigate. `push` takes a route string or an object,
allowing for more options.

### Route parameters
Define a dynamic route segment by prepending it with a colon.

```javascript
const routes = [{ path: '/user/:id', component: User }]
```
The `id` parameter will now be available at `this.$route.params.id`.

Be aware that, because of component caching, the id value will only be
populated once. Watch the `$route` property to get updates properly.
In **vue-router** v2.2 and up, you can pass route parameters as properties,
which eliminates the need for **watch** demonstrated below.

```javascript
export default {
  data(){
    return {
      id: this.$route.params.id
    }
  },
  watch: {
    '$route'(to, from){
      this.id = to.params.id
    }
  }
}
```

### Child routes
Nest child routes in the `children` property on another route.

```javascript
const routes = [
  { path: '', component: Home },
  { path: '/user/:id/edit', component: User, children: [
    { path: '', component: MyComponent },
    { path: 'other', component: MyOtherComponent }
  ]}
];
```

### Named routes
For more convenient linking, add a name property to a route you want to link
to.

```javascript
// in a routes definition...
{ path: '/user/:id/edit', component: UserEdit, name: 'userEdit' }
```

```html
<router-link :to="{ name: 'userEdit', params: { id: 'some-dynamic-id'} }"></router-link>
```

### Query string parameters

Will be appended to the `query` property of `$route`.

```html
<router-link :to="{ name: 'someRoute', query: { foo: 'bar', lang: 'EN'} }"></router-link>
```

```javascript
// in a VM
const lang = this.$route.query.lang; // EN
const foo = this.$route.query.foo; // bar
```

### Named router views
In situation where you want page layout to be different for different routes,
use [**named views**](http://router.vuejs.org/en/essentials/named-views.html)

```javascript
// example
```

### Redirecting
Use the `redirect` property on a route object.

```javascript
const routes = [
  { path: '/', component: Home },
  { path: '/user', component: User },
  { path '/redirect', redirect: '/user' }, // specific redirect
  { path: '*', redirect: '/' } // catch-all route
]
```

### Animating route transitions

```html
<transition name="my-transition-name" mode="out-in">
  <router-view></router-view>
</transition>
```

### Hash changes
Add the `hash` property to a `router-link`'s `to` attribute.

To animate navigation to a hash location on the page, add the `scrollBehavior`
method to  `vue-router` instantiation options.

```javascript
const router = new VueRouter({
  // ..trucated..
  scrollBehavior(to, from, savedPosition){
    // based on selector
    return {
      selector: to.hash
    }
    // or to the last saved position, if there is any
    if(savedPosition){
      return savedPosition;
    }
    // fallback
    return { // hard-coded
      x: 0,
      y: 0
    },
  }
});
```

### beforeEnter / beforeLeave
Execute code before a route is handled. The **global** foreach is executed
before each routs in the application and is defined on the `router` instance.

#### Global

```javascript
router.beforeEach((to, from, next) => {
  console.log('global: before routing');
  next();
});
```

#### Local

The local route intercept option is defined for a specific route definition.

```javascript
const userRoute = { path: '/user', component: User, beforeEnter: (to, from, next) => {
  console.log('local: before enter');
  next();
}},
```

#### Component level
Use the component lifecycle hook `beforeRouteEnter` in a component VM.
Note that you cannot access the `data` property from inside this hook function.

### Lazy loading routes
Load components on demand, instead of all at once, to leverage code splitting
and possibly http2. Webpack's `require.ensure` is used to keep certain modules
out of you main bundle.

```javascript
const User = resolve => {
  require.ensure(['./src/User.vue'], () => {
    resolve(require('./src/User.vue'));
  }, 'group-id')
}
```

If a **group-id** is specified (must be a string), you can group related
components together to form a single bundle, which gets loaded once.

## Vuex
Keep application state in a central store.
For vue applications, it's idiomatic to store vuex related files in a `store`
directory, at the same level as your `components` folder.

```javascript
// in store/store.js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export const store = new Vuex.Store({
  state: {
    counter: 0
  }
});

// in main.js
import { store } from './store/store';

new Vue({
  el:'#app',
  store, // register store to your root Vue instance
  // ..truncated..
})
```

```html
<!-- simple counter example -->
<template>
  <div>
    <h1>Vuex</h1>
    <p>{{ counter }}</p>
    <button @click="increment">increment</button>
    <button @click="decrement">decrement</button>
  </div>
</template>
<script>
export default {
  computed: {
    counter(){
      return this.$store.state.counter;
    }
  },
  methods: {
    increment(){
      this.$store.state.counter++;
    },
    decrement(){
      this.$store.state.counter--;
    }
  }
}
</script>
```

### Getters
Getters are like computed properties for your store. They allow you to keep
manipulations on read operations DRY.

```javascript
export const store = new Vuex.Store({
  state: {
    counter: 0
  },
  getters: {
    double(state){ // all getters receive state as argument.
      return state.counter * 2;
    }
  }
});
```

`vuex` provides the `mapGetters` helper function. This function creates
computed properties on a component for the getters you specify.

Keep in mind that you might need an additional babel plugin to be able to use
`Object.spread` notation.

```html
<script>
  import { mapGetters } from 'vuex';
  export default {
    computed: {
      ...mapGetters([
        'double', // you can now use this vuex getter as computed property in your template
        '...' // and so on
      ]),
      anotherComputed(){
        // code here
      }
    }
  }
</script>
```

### Mutations
Change state.

```javascript
// in store.js
export const store = new Vuex.Store({
  state: {
    counter: 0
  },
  mutations: {
    increment(state){
      state.counter++;
    },
    decrement(state){
      state.counter--;
    }
  }
});

// in component script
import { mapMutations } from 'vuex';
{
  methods: {
    increment(){
      this.$store.commit('increment');
    },
    decrement(){
      this.$store.commit('decrement');
    },
    // note you can also use the helper `mapMutations` here in methods.
    ...mapMutations([
      'increment',
      'decrement'
    ])
  }
}
```

This is all well and good, but all these mutations are required to run
synchronously, if you want to avoid race conditions.

### Actions

To be able to queue tasks that run asynchronously, use **actions**.

Actions get passed a `context` object, of which you can call the `commit`
method after a (potentially) asynchronous task has completed.

```javascript
// in store.js
export const store = new Vuex.Store({
  mutations: {
    increment: // ...
  },
  actions: {
    increment(context) {
      setTimeout(() => { // do something after a while
        context.commit('increment');
      }, 100)
    }
  }
});
// in component script
import { mapActions } from 'vuex';
export default {
  methods: {
    ...mapActions([
      'increment'
    ]),
    // possibly other methods here ...
  }
}
```

Actions you have defined, receive an optional **payload** as second argument.

### Vuex and `v-model`

Wire up v-model to your store by using a writable computed property.

```javascript
// in store.js
export const store = new Vuex.Store({
  state: {
    value: 'hoi'
  },
  getters: {
    value(state){
      return state.value;
    }
  },
  mutations: {
    value(state, payload){
      state.value = payload;
    }
  },
  actions: {
    updateValue({commit}, payload){
      commit('value', payload);
    }
  }
});
```
```html
<template>
  <div>
    <h1>Vuex</h1>
    <input v-model="value">
    <p>{{ value }}</p>
  </div>
</template>
<script>
export default {
  computed: {
    value: {
      get(){
        return this.$store.getters.value;
      },
      set(value){
        this.$store.dispatch('updateValue', value)
      }
    }
  }
}
</script>
```
