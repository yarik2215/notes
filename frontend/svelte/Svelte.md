# [[Svelte]]
Svelte is a tool for building fast web applications.

[Svelte](https://svelte.dev/docs/introduction)
[SvelteKit](https://kit.svelte.dev/docs/introduction)

## Component
A component is a reusable self-contained block of code that encapsulates HTML, CSS and JavaScript that belong together, written into a .svelte file.

```svelte
<script>
    let name = "some";
</script>

<p>This is a {name}.</p>

<style>
	/* Style are scoped to component */
	p {
    color: purple;
    font-family: 'Comic Sans MS', cursive;
    font-size: 2em;
  }
</style>

```

We can import and use other components inside components:

```svelte
<script>
  import Nested from './Nested.svelte';
</script>

<Nested />
```

Add event handler:
``` svelte
<script>
	let count = 0;
	function incrementCount() {
		count += 1;
	}
</script>

<button on:click={incrementCount}>
```

Sync components with reactive declaration:

```js
let count = 0;
$: doubled = count * 2;
```

We can also run arbitrary statements reactively and group statements together with a block:
```js
$: {
  console.log('the count is ' + count);
  alert('I SAID THE COUNT IS ' + count);
}
```
Or using `if`:
```js
$: if (count >= 10) {
  alert('count is dangerously high!');
  count = 9;
}
```

> Svelte's reactivity is triggered by assignments. Methods that mutate arrays or objects will not trigger updates by themselves. One way to fix that is to assign numbers to itself to tell the compiler it has changed. The same rule applies to array methods such as pop, shift, and splice and to object methods such as Map.set, Set.add, etc.

## Properties
In Svelte, we create props with the export keyword.

```svelte
<script>
  export let answer;
</script>
```
> We can easily specify default values for props.

If you have an object of properties, you can 'spread' them onto a component instead of specifying each one:

```svelte
<Info {...pkg} />
```

> Conversely, if you need to reference all the props that were passed into a component, including ones that weren't declared with export, you can do so by accessing $$props directly.

## Condition expressions
To conditionally render some markup, we wrap it in an `if` block:

```svelte
{#if x > 10}
  <p>{x} is greater than 10</p>
{:else if 5 > x}
  <p>{x} is less than 5</p>
{:else}
  <p>{x} is between 5 and 10</p>
{/if}
```

If you need to loop over lists of data, use an `each` block:

```svelte
<script>
	let cats = [...]
</script>

<ul>
  {#each cats as cat}
    <li>{cat.name}</li>
  {/each}
</ul>
```

>The expression (cats, in this case) can be any array or array-like object (i.e. it has a length property). You can loop over generic iterables with each [...iterable].

You can get the current index as a second argument, like so:

```svelte
{#each cats as cat, i}
```

> If you prefer, you can use destructuring — each cats as { id, name }

To bound some id to DOM node, we specify a unique identifier (or "key") for the each block:

```svelte
{#each things as thing (thing.id)}
  <Thing name={thing.name} />
{/each}
```

Here, `(thing.id)` is the key, which tells Svelte how to figure out which DOM node to change when the component updates.

## Working with promises

Most web applications have to deal with asynchronous data at some point. Svelte makes it easy to await the value of promises directly in your markup:

```svelte
{#await promise}
  <p>...waiting</p>
{:then number}
  <p>The number is {number}</p>
{:catch error}
  <p style="color: red">{error.message}</p>
{/await}
```

>Only the most recent promise is considered, meaning you don't need to worry about race conditions.

If you know that your promise can't reject, you can omit the catch block. You can also omit the first block if you don't want to show anything until the promise resolves:

```svelte
{#await promise then number}
  <p>the number is {number}</p>
{/await}
```

## Events

As we've briefly seen already, you can listen to any event on an element with the on: directive:

```svelte
<div on:mousemove={handleMousemove}>
  The mouse position is {m.x} x {m.y}
</div>
```

You can also declare event handlers inline:

```svelte
<div on:mousemove={(e) => (m = { x: e.clientX, y: e.clientY })}>
  The mouse position is {m.x} x {m.y}
</div>
```

### Modifiers

DOM event handlers can have modifiers that alter their behaviour. For example, a handler with a once modifier will only run a single time:

The full list of modifiers:

```svelte
<button on:click|once={handleClick}> Click me </button>
```

- `preventDefault` — calls event.preventDefault() before running the handler. Useful for client-side form handling, for example.
- `stopPropagation` — calls event.stopPropagation(), preventing the event reaching the next element
- `passive` — improves scrolling performance on touch/wheel events (Svelte will add it automatically where it's safe to do so)
- `nonpassive` — explicitly set passive: false
- `capture` — fires the handler during the capture phase instead of the bubbling phase ([MDN docs](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#Event_bubbling_and_capture))
- `once` — remove the handler after the first time it runs
- `self` — only trigger handler if event.target is the element itself
- `trusted` — only trigger handler if event.isTrusted is true. I.e. if the event is triggered by a user action.

> You can chain modifiers together, e.g. on:click|once|capture={...}.

### Component events

Components can also dispatch events. To do so, they must create an event dispatcher.

```svelte
<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  function sayHello() {
    dispatch('message', {
      text: 'Hello!'
    });
  }
</script>
```

We can listen to this event then using `on:` followed by the event name that we are dispatching (in this case, message) -> `on:message`.

> `createEventDispatcher` must be called when the component is first instantiated — you can't do it later inside e.g. a setTimeout callback. This links dispatch to the component instance.

### Event forwarding

Unlike DOM events, component events don't bubble. If you want to listen to an event on some deeply nested component, the intermediate components must forward the event.
Svelte gives us an equivalent shorthand — an on:message event directive without a value means 'forward all message events'.

```svelte
<script>
  import Inner from './Inner.svelte';
</script>

<Inner on:message />
```

> Event forwarding works for DOM events too. We want to get notified of clicks on our `<CustomButton>` — to do that, we just need to forward click events on the `<button>` element in CustomButton.svelte.

## Bindings

As a general rule, data flow in Svelte is top down — a parent component can set props on a child component, and a component can set attributes on an element, but not the other way around.

Sometimes it's useful to break that rule. Take the case of the `<input>` element in this component — we could add an on:input event handler that sets the value of name to event.target.value, but it's a bit... boilerplatey. It gets even worse with other form elements, as we'll see.

Instead, we can use the bind:value directive:

```svelte
<input bind:value={name} />
```

This means that not only will changes to the value of name update the input value, but changes to the input value will update name.

In the DOM, everything is a string. That's unhelpful when you're dealing with numeric inputs — type="number" and type="range" — as it means you have to remember to coerce input.value before using it.

With bind:value, Svelte takes care of it for you:

``` svelte
<input type="number" bind:value={a} min="0" max="10" />
<input type="range" bind:value={a} min="0" max="10" />
```

Checkboxes are used for toggling between states. Instead of binding to input.value, we bind to input.checked:

```svelte
<input type="checkbox" bind:checked={yes} />
```

### Group inputs

If you have multiple inputs relating to the same value, you can use bind:group along with the value attribute. Radio inputs in the same group are mutually exclusive; checkbox inputs in the same group form an array of selected values.

```svelte
<input type="radio" bind:group={scoops} name="scoops" value={1} />
```

We can also use bind:value with `<select>` elements.

```svelte
<form on:submit|preventDefault={handleSubmit}>
	<select bind:value={selected} on:change={() => (answer = '')}>
		{#each questions as question}
			<option value={question}>
				{question.text}
			</option>
		{/each}
	</select>

	<input bind:value={answer} />

	<button disabled={!answer} type="submit"> Submit </button>
</form>
```

A select can have a multiple attribute, in which case it will populate an array rather than selecting a single value.

Elements with the `contenteditable` attribute support the following bindings:

- `innerHTML`
- `innerText`
- `textContent`

There are slight differences between each of these, read more about them [here](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent#Differences_from_innerText).

```svelte
<div contenteditable="true" bind:innerHTML={html} />
```

You can even bind to properties inside an each block.

```svelte
{#each todos as todo}
  <div class:done={todo.done}>
    <input type="checkbox" bind:checked={todo.done} />

    <input placeholder="What needs to be done?" bind:value={todo.text} />
  </div>
{/each}
```

> Note that interacting with these `<input>` elements will mutate the array. If you prefer to work with immutable data, you should avoid these bindings and use event handlers instead.

Passing arguments into handler. If we want to pass additional info into event handler we need to create a wrapper on top of it.

```svelte
{#each todos as todo, i}
  <div class:done={todo.done}>
    ...
    <button on:click={() => deleteTodo(i)}>Delete</button>
  </div>
{/each}
```

### Media elements

The `<audio>` and `<video>` elements have several properties that you can bind to. See [tutorial](https://svelte.dev/tutorial/media-elements) for more details.

### Dimensions

Every block-level element has clientWidth, clientHeight, offsetWidth and offsetHeight bindings. These bindings are readonly — changing the values of w and h won't have any effect.

### Binding This

The readonly `this` binding applies to every element (and component) and allows you to obtain a reference to rendered elements. For example, we can get a reference to a <canvas> element:

```svelte
<canvas bind:this={canvas} width={32} height={32} />
```

Note that the value of canvas will be undefined until the component has mounted, so we put the logic inside the `onMount` lifecycle function.

### Component bindings

Just as you can bind to properties of DOM elements, you can bind to component props.
Just as you can bind to DOM elements, you can bind to component instances themselves.

## Lifecycle

Every component has a lifecycle that starts when it is created, and ends when it is destroyed. There are a handful of functions that allow you to run code at key moments during that lifecycle.

### onMount

`onMount` runs after the component is first rendered to the DOM. 

```svelte
<script>
  import { onMount } from 'svelte';
  let photos = [];
  onMount(async () => {
    const res = await fetch(`/tutorial/api/album`);
    photos = await res.json();
  });
</script>
```

### onDestroy

To run code when your component is destroyed, use `onDestroy`.

```svelte
<script>
  import { onDestroy } from 'svelte';
  let counter = 0;
  const interval = setInterval(() => (counter += 1), 1000);
  onDestroy(() => clearInterval(interval));
</script>
```

### Before and after Update

The `beforeUpdate` function schedules work to happen immediately before the DOM is updated. `afterUpdate` is its counterpart, used for running code once the DOM is in sync with your data.

### Tick

The `tick` function is unlike other lifecycle functions in that you can call it any time, not just when the component first initialises. It returns a promise that resolves as soon as any pending state changes have been applied to the DOM (or immediately, if there are no pending state changes).
When you update component state in Svelte, it doesn't update the DOM immediately. Instead, it waits until the next microtask to see if there are any other changes that need to be applied, including in other components. Doing so avoids unnecessary work and allows the browser to batch things more effectively.

## Stores

### Writable store

A store is simply an object with a subscribe method that allows interested parties to be notified whenever the store value changes. 
```svelte
<script>
  // create store
  import { writable } from 'svelte/store';
  export const count = writable(0);
  // set store value
  counter.set(0)
  // update value in store
  count.update((n) => n + 1);
  // subscribe to a value
  count.subscribe((value) => {
    countValue = value;
  });
</script>
```

### AutoSubscriptions

> The app in the previous example works, but there's a subtle bug — the store is subscribed to, but never unsubscribed. If the component was instantiated and destroyed many times, this would result in a memory leak.

Start by declaring unsubscribe in App.svelte:
```js
const unsubscribe = count.subscribe((value) => {
  countValue = value;
});
```

> Calling a subscribe method returns an unsubscribe function.

Svelte has a trick up its sleeve — you can reference a store value by prefixing the store name with `$`:
```svelte
<script>
  import { count } from './stores.js';
</script>

<h1>The count is {$count}</h1>
```

> Auto-subscription only works with store variables that are declared (or imported) at the top-level scope of a component.

### Readable store

The first argument to readable is an initial value, which can be null or undefined if you don't have one yet. The second argument is a start function that takes a set callback and returns a stop function. The start function is called when the store gets its first subscriber; stop is called when the last subscriber unsubscribes.

```js
export const time = readable(new Date(), function start(set) {
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);
 
  return function stop() {
    clearInterval(interval);
  };
});
```

### Derived store

You can create a store whose value is based on the value of one or more other stores with `derived`.

```js
export const elapsed = derived(time, ($time) => Math.round(($time - start) / 1000));
```

> It's possible to derive a store from multiple inputs, and to explicitly set a value instead of returning it (which is useful for deriving values asynchronously).

### Custom stores

As long as an object correctly implements the subscribe method, it's a store. Beyond that, anything goes. It's very easy, therefore, to create custom stores with domain-specific logic.
Example with counter store:

```js
function createCount() {
  const { subscribe, set, update } = writable(0);
 
  return {
    subscribe,
    increment: () => update((n) => n + 1),
    decrement: () => update((n) => n - 1),
    reset: () => set(0)
  };
}
```

### Store bindings
If a store is writable — i.e. it has a set method — you can bind to its value, just as you can bind to local component state.

```svelte
<input bind:value={$name} />
```
