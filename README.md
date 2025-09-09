# Svelte Cheat Sheet

A comprehensive reference for Svelte - a radical new approach to building user interfaces with compile-time optimizations.

---

## Table of Contents
- [Basic Setup](#basic-setup)
- [Component Basics](#component-basics)
- [Reactivity](#reactivity)
- [Props and Events](#props-and-events)
- [Stores](#stores)
- [Transitions](#transitions)
- [Actions](#actions)
- [Routing](#routing)
- [SvelteKit](#sveltekit)

---

## Basic Setup

```bash
# Create new project
npm create svelte@latest my-app
cd my-app
npm install
npm run dev

# With SvelteKit
npm create svelte@latest my-kit-app
```

## Component Basics

### Basic Component Structure
```svelte
<!-- App.svelte -->
<script>
  let name = 'world';
  let count = 0;
  
  function increment() {
    count += 1;
  }
  
  $: doubled = count * 2;
  $: if (count >= 10) {
    alert('Count is getting high!');
  }
</script>

<h1>Hello {name}!</h1>
<button on:click={increment}>
  Count: {count}
</button>
<p>Doubled: {doubled}</p>

<style>
  h1 {
    color: purple;
  }
</style>
```

### Conditional Rendering
```svelte
<script>
  let user = { loggedIn: false };
  let x = 7;
</script>

{#if user.loggedIn}
  <button on:click={logout}>Log out</button>
{:else}
  <button on:click={login}>Log in</button>
{/if}

{#if x > 10}
  <p>{x} is greater than 10</p>
{:else if 5 > x}
  <p>{x} is less than 5</p>
{:else}
  <p>{x} is between 5 and 10</p>
{/if}
```

### Loops
```svelte
<script>
  let cats = [
    { id: 'J---aiyznGQ', name: 'Keyboard Cat' },
    { id: 'z_AbfPXTKms', name: 'Maru' },
    { id: 'OUtn3pvWmpg', name: 'Henri The Existential Cat' }
  ];
</script>

<ul>
  {#each cats as cat, i}
    <li>{i + 1}: {cat.name}</li>
  {/each}
</ul>

<!-- With keyed each blocks -->
{#each cats as cat (cat.id)}
  <div>
    <h2>{cat.name}</h2>
  </div>
{/each}

<!-- With destructuring -->
{#each cats as { id, name }, i (id)}
  <div>{i}: {name}</div>
{/each}
```

## Reactivity

### Reactive Statements
```svelte
<script>
  let count = 0;
  
  // Reactive declaration
  $: doubled = count * 2;
  
  // Reactive statement
  $: if (count >= 10) {
    alert('Count is getting high!');
  }
  
  // Reactive block
  $: {
    console.log(`The count is ${count}`);
    document.title = `Count: ${count}`;
  }
  
  let numbers = [1, 2, 3, 4];
  
  // Depends on array contents, not array itself
  $: sum = numbers.reduce((t, n) => t + n, 0);
  
  function addNumber() {
    // This will trigger reactivity
    numbers = [...numbers, numbers.length + 1];
  }
  
  let obj = { foo: { bar: 1 } };
  
  function updateNestedProperty() {
    // This won't trigger reactivity
    obj.foo.bar = 2;
    
    // This will trigger reactivity
    obj = obj;
    // or
    obj = { ...obj, foo: { ...obj.foo, bar: 2 } };
  }
</script>
```

### Event Handling
```svelte
<script>
  let m = { x: 0, y: 0 };
  
  function handleMousemove(event) {
    m.x = event.clientX;
    m.y = event.clientY;
  }
  
  function handleClick(event) {
    console.log('Button clicked!');
  }
</script>

<div on:mousemove={handleMousemove}>
  The mouse position is {m.x} x {m.y}
</div>

<!-- Event modifiers -->
<button on:click|once={handleClick}>
  Click me (only works once)
</button>

<form on:submit|preventDefault={handleSubmit}>
  <!-- ... -->
</form>

<button on:click|stopPropagation={handleClick}>
  Click me
</button>

<!-- Event forwarding -->
<button on:click>
  This will forward the click event
</button>
```

## Props and Events

### Props
```svelte
<!-- Nested.svelte -->
<script>
  export let answer = 42;
  export let name;
  
  // Props can have default values
  export let version = '1.0.0';
  
  // You can also spread props
  export let className = '';
</script>

<p>The answer is {answer}</p>
<p>Hello {name}!</p>

<!-- Using the component -->
<!-- App.svelte -->
<script>
  import Nested from './Nested.svelte';
  
  let props = {
    answer: 42,
    name: 'world'
  };
</script>

<Nested answer={42} name="world" />
<Nested {...props} />
```

### Custom Events
```svelte
<!-- Child.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  const dispatch = createEventDispatcher();
  
  function sayHello() {
    dispatch('message', {
      text: 'Hello!'
    });
  }
  
  function forward() {
    // Forward DOM events
    dispatch('click');
  }
</script>

<button on:click={sayHello}>
  Say hello
</button>

<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';
  
  function handleMessage(event) {
    alert(event.detail.text);
  }
</script>

<Child on:message={handleMessage} />

<!-- Or inline -->
<Child on:message={event => alert(event.detail.text)} />
```

### Slots
```svelte
<!-- Card.svelte -->
<div class="card">
  <header>
    <slot name="header"></slot>
  </header>
  
  <main>
    <slot></slot>
  </main>
  
  <footer>
    <slot name="footer">
      <!-- Default content -->
      <p>Copyright 2023</p>
    </slot>
  </footer>
</div>

<!-- App.svelte -->
<Card>
  <h1 slot="header">Card Title</h1>
  
  <p>This is the main content.</p>
  
  <div slot="footer">
    <button>Action</button>
  </div>
</Card>
```

## Stores

### Writable Stores
```javascript
// stores.js
import { writable } from 'svelte/store';

export const count = writable(0);

export const user = writable({
  name: 'Guest',
  email: null
});

// Custom store
function createCounter() {
  const { subscribe, set, update } = writable(0);
  
  return {
    subscribe,
    increment: () => update(n => n + 1),
    decrement: () => update(n => n - 1),
    reset: () => set(0)
  };
}

export const counter = createCounter();
```

```svelte
<!-- Component using stores -->
<script>
  import { count, counter, user } from './stores.js';
  
  // Auto-subscription with $
  console.log($count);
  
  // Manual subscription
  const unsubscribe = count.subscribe(value => {
    console.log(value);
  });
  
  // Don't forget to unsubscribe
  import { onDestroy } from 'svelte';
  onDestroy(unsubscribe);
  
  function increment() {
    count.update(n => n + 1);
  }
</script>

<h1>Count: {$count}</h1>
<button on:click={increment}>+</button>
<button on:click={counter.increment}>Custom +</button>
<button on:click={counter.decrement}>Custom -</button>
<button on:click={counter.reset}>Reset</button>

<p>User: {$user.name}</p>
```

### Readable Stores
```javascript
// stores.js
import { readable } from 'svelte/store';

export const time = readable(new Date(), function start(set) {
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);
  
  return function stop() {
    clearInterval(interval);
  };
});
```

### Derived Stores
```javascript
// stores.js
import { derived, writable } from 'svelte/store';

export const name = writable('world');

export const greeting = derived(
  name,
  $name => `Hello ${$name}!`
);

// Multiple stores
export const a = writable(1);
export const b = writable(2);

export const sum = derived([a, b], ([a, b]) => a + b);

// Async derived
export const users = derived(
  selectedUserId,
  async ($selectedUserId, set) => {
    if ($selectedUserId) {
      const response = await fetch(`/users/${$selectedUserId}`);
      const user = await response.json();
      set(user);
    }
  },
  null // Initial value
);
```

## Transitions

### Built-in Transitions
```svelte
<script>
  import { fade, blur, fly, slide, scale } from 'svelte/transition';
  import { quintOut } from 'svelte/easing';
  
  let visible = true;
</script>

<label>
  <input type="checkbox" bind:checked={visible}>
  visible
</label>

{#if visible}
  <p transition:fade>
    Fades in and out
  </p>
  
  <p transition:fly="{{ y: 200, duration: 2000 }}">
    Flies in and out
  </p>
  
  <p 
    in:fly="{{ y: 200, duration: 2000 }}"
    out:fade="{{ duration: 200 }}">
    Different in/out transitions
  </p>
  
  <p transition:scale="{{ duration: 500, delay: 500, opacity: 0.5, start: 0.5, easing: quintOut }}">
    Scales with custom options
  </p>
{/if}
```

### Custom Transitions
```javascript
// transitions.js
export function typewriter(node, { speed = 1 }) {
  const valid = (
    node.childNodes.length === 1 &&
    node.childNodes[0].nodeType === Node.TEXT_NODE
  );
  
  if (!valid) {
    throw new Error(`This transition only works on elements with a single text node child`);
  }
  
  const text = node.textContent;
  const duration = text.length / (speed * 0.01);
  
  return {
    duration,
    tick: t => {
      const i = Math.trunc(text.length * t);
      node.textContent = text.slice(0, i);
    }
  };
}
```

```svelte
<script>
  import { typewriter } from './transitions.js';
  
  let visible = false;
</script>

{#if visible}
  <p transition:typewriter>
    The quick brown fox jumps over the lazy dog
  </p>
{/if}
```

## Actions

### Built-in Actions
```svelte
<script>
  let text = '';
  
  function selectOnFocus(node) {
    const handleFocus = event => {
      node.select();
    }
    
    node.addEventListener('focus', handleFocus);
    
    return {
      destroy() {
        node.removeEventListener('focus', handleFocus);
      }
    };
  }
</script>

<input bind:value={text} use:selectOnFocus />
```

### Custom Actions
```javascript
// actions.js
export function clickOutside(node, callback) {
  const handleClick = event => {
    if (node && !node.contains(event.target) && !event.defaultPrevented) {
      callback();
    }
  }
  
  document.addEventListener('click', handleClick, true);
  
  return {
    destroy() {
      document.removeEventListener('click', handleClick, true);
    }
  };
}

export function tooltip(node, text) {
  let tooltipElement;
  
  function showTooltip() {
    tooltipElement = document.createElement('div');
    tooltipElement.textContent = text;
    tooltipElement.className = 'tooltip';
    document.body.appendChild(tooltipElement);
  }
  
  function hideTooltip() {
    if (tooltipElement) {
      tooltipElement.remove();
      tooltipElement = null;
    }
  }
  
  node.addEventListener('mouseenter', showTooltip);
  node.addEventListener('mouseleave', hideTooltip);
  
  return {
    update(newText) {
      text = newText;
      if (tooltipElement) {
        tooltipElement.textContent = text;
      }
    },
    destroy() {
      node.removeEventListener('mouseenter', showTooltip);
      node.removeEventListener('mouseleave', hideTooltip);
      hideTooltip();
    }
  };
}
```

```svelte
<script>
  import { clickOutside, tooltip } from './actions.js';
  
  let showModal = false;
</script>

{#if showModal}
  <div class="modal" use:clickOutside={() => showModal = false}>
    <p>Modal content</p>
  </div>
{/if}

<button use:tooltip={'Helpful tooltip text'}>
  Hover me
</button>
```

## Routing

### Page.js Router
```bash
npm install page
```

```javascript
// router.js
import router from 'page';
import Home from './Home.svelte';
import About from './About.svelte';
import Contact from './Contact.svelte';

export let page;
export let params = {};

router('/', () => page = Home);
router('/about', () => page = About);
router('/contact', () => page = Contact);
router('/users/:id', (ctx) => {
  params = ctx.params;
  page = UserDetail;
});

router.start();
```

```svelte
<!-- App.svelte -->
<script>
  import './router.js';
  import { page, params } from './router.js';
</script>

<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>

<main>
  <svelte:component this={page} {params} />
</main>
```

## SvelteKit

### Basic Setup
```bash
npm create svelte@latest my-app
cd my-app
npm install
npm run dev
```

### File-based Routing
```
src/routes/
├── +layout.svelte
├── +page.svelte
├── about/
│   └── +page.svelte
├── blog/
│   ├── +page.svelte
│   └── [slug]/
│       └── +page.svelte
└── api/
    └── posts/
        └── +server.js
```

### Page and Layout
```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import '../app.css';
</script>

<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/blog">Blog</a>
</nav>

<main>
  <slot />
</main>

<style>
  nav {
    padding: 1rem;
    border-bottom: 1px solid #ccc;
  }
</style>
```

```svelte
<!-- src/routes/+page.svelte -->
<script>
  export let data;
</script>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

{#if data.posts}
  <h2>Latest Posts</h2>
  {#each data.posts as post}
    <article>
      <h3><a href="/blog/{post.slug}">{post.title}</a></h3>
      <p>{post.excerpt}</p>
    </article>
  {/each}
{/if}
```

### Load Functions
```javascript
// src/routes/+page.js
export async function load({ fetch }) {
  const response = await fetch('/api/posts');
  const posts = await response.json();
  
  return {
    posts
  };
}

// src/routes/blog/[slug]/+page.js
export async function load({ params, fetch }) {
  const response = await fetch(`/api/posts/${params.slug}`);
  
  if (response.ok) {
    const post = await response.json();
    return { post };
  }
  
  throw error(404, 'Not found');
}
```

### API Routes
```javascript
// src/routes/api/posts/+server.js
import { json } from '@sveltejs/kit';

export async function GET() {
  const posts = [
    { id: 1, title: 'Hello World', slug: 'hello-world', excerpt: 'First post' },
    { id: 2, title: 'Svelte', slug: 'svelte', excerpt: 'About Svelte' }
  ];
  
  return json(posts);
}

export async function POST({ request }) {
  const data = await request.json();
  
  // Save post logic here
  
  return json({ success: true }, { status: 201 });
}

// src/routes/api/posts/[id]/+server.js
export async function GET({ params }) {
  const { id } = params;
  
  // Fetch post by id
  const post = await getPost(id);
  
  if (post) {
    return json(post);
  }
  
  throw error(404, 'Post not found');
}
```

### Forms
```svelte
<!-- src/routes/contact/+page.svelte -->
<script>
  import { enhance } from '$app/forms';
  
  export let form;
</script>

<h1>Contact</h1>

<form method="POST" use:enhance>
  <label>
    Name
    <input name="name" type="text" required />
  </label>
  
  <label>
    Email
    <input name="email" type="email" required />
  </label>
  
  <label>
    Message
    <textarea name="message" required></textarea>
  </label>
  
  <button type="submit">Send</button>
</form>

{#if form?.success}
  <p>Thank you for your message!</p>
{/if}

{#if form?.errors}
  <div class="errors">
    {#each Object.entries(form.errors) as [field, error]}
      <p>{field}: {error}</p>
    {/each}
  </div>
{/if}
```

```javascript
// src/routes/contact/+page.server.js
import { fail } from '@sveltejs/kit';

export const actions = {
  default: async ({ request }) => {
    const data = await request.formData();
    const name = data.get('name');
    const email = data.get('email');
    const message = data.get('message');
    
    // Validation
    if (!name || !email || !message) {
      return fail(400, {
        error: 'All fields are required',
        name,
        email,
        message
      });
    }
    
    // Send email or save to database
    
    return { success: true };
  }
};
```

---

## Best Practices

### Component Organization
```svelte
<script>
  // 1. Imports
  import { onMount } from 'svelte';
  import Child from './Child.svelte';
  
  // 2. Exports (props)
  export let prop1;
  export let prop2 = 'default';
  
  // 3. Local variables
  let localVar = '';
  
  // 4. Reactive declarations
  $: computed = prop1 + prop2;
  
  // 5. Functions
  function handleClick() {
    // ...
  }
  
  // 6. Lifecycle
  onMount(() => {
    // ...
  });
</script>

<!-- Template -->
<div>
  <!-- Content -->
</div>

<!-- Styles last -->
<style>
  div {
    /* styles */
  }
</style>
```

---

## Resources
- [Official Svelte Tutorial](https://svelte.dev/tutorial)
- [Svelte Documentation](https://svelte.dev/docs)
- [SvelteKit Documentation](https://kit.svelte.dev/docs)
- [Svelte Examples](https://svelte.dev/examples)
- [Svelte REPL](https://svelte.dev/repl)

---
*Originally compiled from various sources. Contributions welcome!*