# vue3-lazy-hydration

> Lazy Hydration of Server-Side Rendered Vue.js v3 Components

Inspired by [`vue-lazy-hydration`](https://github.com/maoberlehner/vue-lazy-hydration), this library brings a renderless component, composables and import wrappers to delay the hydration of pre-rendered HTML.

## Installation

Use the package manager [yarn v1](https://classic.yarnpkg.com/) or [npm](https://github.com/npm/cli) to install vue3-lazy-hydration.

```bash
# install with yarn
yarn add vue3-lazy-hydration

# install with npm
npm install vue3-lazy-hydration
```

Optionally make the renderless component [available globally](https://vuejs.org/guide/components/registration.html#global-registration).

```js
import { createSSRApp } from 'vue';
import { LazyHydrationWrapper } from 'vue3-lazy-hydration';

const app = createSSRApp({});

app.component(
  // custom registered name
  'LazyHydrate',
  LazyHydrationWrapper
);
```

## Usage

### Renderless Component

- Never hydrate.

  ```html
  <template>
    <LazyHydrationWrapper @hydrated="onHydrated">
      <!--
        Content never hydrated.
      -->
    </LazyHydrationWrapper>
  </template>

  <script setup>
    import { LazyHydrationWrapper } from 'vue3-lazy-hydration';

    function onHydrated() {
      console.log('this function will never be called !');
    }
  </script>
  ```

- Delays hydration until the browser is idle.

  ```html
  <template>
    <LazyHydrationWrapper :when-idle="4000" @hydrated="onHydrated">
      <!--
      Content hydrated when the browser is idle, or when the timeout
        of 4000ms has elapsed and hydration has not already taken place.
      -->
    </LazyHydrationWrapper>
  </template>
  <script setup>
    import { LazyHydrationWrapper } from 'vue3-lazy-hydration';

    function onHydrated() {
      console.log('content hydrated !');
    }
  </script>
  ```

- Delays hydration until one of the root elements is visible.

  ```html
  <template>
    <LazyHydrationWrapper
      :when-visible="{ rootMargin: '50px' }"
      @hydrated="onHydrated"
    >
      <!--
      Content hydrated when one of the root elements is visible.
      All root elements are observed with a margin of 50px.
      -->
    </LazyHydrationWrapper>
  </template>

  <script setup>
    import { LazyHydrationWrapper } from 'vue3-lazy-hydration';

    function onHydrated() {
      console.log('content hydrated !');
    }
  </script>
  ```

- Delays hydration until one of the elements triggers a DOM event (focus by default).

  ```html
  <template>
    <LazyHydrationWrapper
      :on-interaction="['click', 'touchstart']"
      @hydrated="onHydrated"
    >
      <!--
      Content hydrated when one of the elements triggers a click or touchstart event.
      -->
    </LazyHydrationWrapper>
  </template>

  <script setup>
    import { LazyHydrationWrapper } from 'vue3-lazy-hydration';

    function onHydrated() {
      console.log('content hydrated !');
    }
  </script>
  ```

- Delays hydration until manually triggered.

  ```html
  <template>
    <button @click="triggerHydration">Trigger hydration</button>

    <LazyHydrationWrapper :when-triggered="triggered" @hydrated="onHydrated">
      <!--
      Content hydrated when the above button is clicked.
      -->
    </LazyHydrationWrapper>
  </template>

  <script setup>
    import { ref } from 'vue';
    import { LazyHydrationWrapper } from 'vue3-lazy-hydration';

    const triggered = ref(false);

    function triggerHydration() {
      triggered.value = true;
    }

    function onHydrated() {
      console.log('content hydrated !');
    }
  </script>
  ```

#### Props declaration

```js
props: {

  /* Number type refers to the timeout option passed to the requestIdleCallback API
  * @see https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback
  */
  whenIdle: {
    default: false,
    type: [Boolean, Number],
  },

  /* Object type refers to the options passed to the IntersectionObserver API
  * @see https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
  */
  whenVisible: {
    default: false,
    type: [Boolean, Object],
  },

  /*
  * @see https://developer.mozilla.org/en-US/docs/Web/API/Element#events
  */
  onInteraction: {
    default: false,
    type: [Array, Boolean, String],
  },

  /*
  * Object type refers to a ref object
  * @see https://vuejs.org/api/reactivity-core.html#ref
  */
  whenTriggered: {
    default: undefined,
    type: [Boolean, Object],
  },
}
```

### Composables

#### `useLazyHydration()`

```html
<script setup>
  import { useLazyHydration } from 'vue3-lazy-hydration';

  // delays hydration until hydrate function is called
  const { willPerformHydration, hydrate, onHydrated, onCleanup } =
    useLazyHydration();

  if (willPerformHydration === false) {
    /**
     * The application is actually running at server-side
     * or subsequent navigation occurred at client-side after the first load.
     */

    return;
  }

  // this hook is run when component is hydrated
  // and all its child asynchronous components are resolved
  onHydrated(() => {
    console.log('content hydrated !');
  });

  // this hook is run just before hydration
  onCleanup(() => {
    console.log('clean side effects (timeout, listeners, etc...)');
  });

  // optionnaly hydrate the component
  hydrate();
</script>
```

#### `useHydrateWhenIdle({ hydrate, onCleanup }, timeout = 2000)`

```html
<script setup>
  import { useLazyHydration, useHydrateWhenIdle } from 'vue3-lazy-hydration';

  // delays hydration
  const { hydrate, onCleanup } = useLazyHydration();

  // hydrate when browser is idle
  // or when the timeout of 4000ms has elapsed
  // and the component has not already been hydrated
  useHydrateWhenIdle({ hydrate, onCleanup }, 4000);
</script>
```

#### `useHydrateWhenVisible({ hydrate, onCleanup }, observerOpts = {})`

```html
<script setup>
  import { useLazyHydration, useHydrateWhenVisible } from 'vue3-lazy-hydration';

  // delays hydration
  const { hydrate, onCleanup } = useLazyHydration();

  // https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
  const observerOptions = {
    rootMargin: '0px',
    threshold: 1.0,
  };

  // hydrate when one of the root elements is visible
  useHydrateWhenVisible({ hydrate, onCleanup }, observerOptions);
</script>
```

#### `useHydrateOnInteraction({ hydrate, onCleanup }, events = ['focus'])`

```html
<script setup>
  import { useLazyHydration, useHydrateOnInteraction } from 'vue3-lazy-hydration';

  // delays hydration
  const { hydrate, onCleanup } = useLazyHydration();

  // hydrate when one of the elements triggers a focus or click event
  useHydrateOnInteraction({ hydrate, onCleanup }, ['focus' 'click']);
</script>
```

#### `useHydrateWhenTriggered({ hydrate, onCleanup }, trigger)`

```html
<script setup>
  import { toRef } from 'vue';
  import {
    useLazyHydration,
    useHydrateWhenTriggered,
  } from 'vue3-lazy-hydration';

  const props = defineProps({
    triggerHydration: {
      default: undefined,
      type: [Boolean, Object],
    },
  });

  // delays hydration
  const { hydrate, onCleanup } = useLazyHydration();

  // trigger hydration when the triggerHydration property changes to true
  useHydrateWhenTriggered(result, toRef(props, 'triggerHydration'));
</script>
```

### Import Wrappers

#### `hydrateNever(component)`

Wrap a component in a renderless component that will never be hydrated.

```html
<script setup>
  import { resolveComponent, defineAsyncComponent } from 'vue';

  import { hydrateNever } from 'vue3-lazy-hydration';

  // wrap a globally registered component resolved by its name
  const NeverHydratedComp = hydrateNever(resolveComponent('ComponentA'));

  // wrap an async component
  const NeverHydratedAsyncComp = hydrateNever(
    defineAsyncComponent(() => import('./ComponentB.vue'))
  );
</script>

<template>
  <NeverHydratedComp />

  <NeverHydratedAsyncComp />
</template>
```

#### `hydrateWhenIdle(component, timeout = 2000)`

Wrap a component in a renderless component that will be hydrated when browser is idle.

```html
<script setup>
  import { resolveComponent, defineAsyncComponent } from 'vue';

  import { hydrateWhenIdle } from 'vue3-lazy-hydration';

  // wrap a globally registered component resolved by its name
  const LazilyHydratedComp = hydrateWhenIdle(
    resolveComponent('ComponentA'),
    2000
  );

  // wrap an async component
  const LazilyHydratedAsyncComp = hydrateWhenIdle(
    defineAsyncComponent(() => import('./ComponentB.vue')),
    4000
  );
</script>

<template>
  <LazilyHydratedComp />

  <LazilyHydratedAsyncComp />
</template>
```

#### `hydrateWhenVisible(component, observerOpts = {})`

Wrap a component in a renderless component that will be hydrated when one of the root elements is visible.

```html
<script setup>
  import { resolveComponent, defineAsyncComponent } from 'vue';

  import { hydrateWhenVisible } from 'vue3-lazy-hydration';

  // wrap a globally registered component resolved by its name
  const LazilyHydratedComp = hydrateWhenVisible(
    resolveComponent('ComponentA'),
    { rootMargin: '50px' }
  );

  // wrap an async component
  const LazilyHydratedAsyncComp = hydrateWhenVisible(
    defineAsyncComponent(() => import('./ComponentB.vue')),
    {
      threshold: [0, 0.25, 0.5, 0.75, 1],
    }
  );
</script>

<template>
  <LazilyHydratedComp />

  <LazilyHydratedAsyncComp />
</template>
```

#### `hydrateOnInteraction(component, events = ['focus'])`

Wrap a component in a renderless component that will be hydrated when one of the elements trigger one of the events in the `events` parameter.

```html
<script setup>
  import { resolveComponent, defineAsyncComponent } from 'vue';

  import { hydrateOnInteraction } from 'vue3-lazy-hydration';

  // wrap a globally registered component resolved by its name
  const LazilyHydratedComp = hydrateOnInteraction(
    resolveComponent('ComponentA')
  );

  // wrap an async component
  const LazilyHydratedAsyncComp = hydrateOnInteraction(
    defineAsyncComponent(() => import('./ComponentB.vue')),
    ['focus', 'click', 'touchstart']
  );
</script>

<template>
  <LazilyHydratedComp />

  <LazilyHydratedAsyncComp />
</template>
```

#### `hydrateWhenTriggered(component, trigger)`

Wrap a component in a renderless component that will be hydrated when the `trigger` parameter changes to true.

```html
<script setup>
  import { ref, resolveComponent, defineAsyncComponent } from 'vue';

  import { hydrateOnInteraction } from 'vue3-lazy-hydration';

  const hydrationTriggered = ref(false);

  // wrap a globally registered component resolved by its name
  const LazilyHydratedComp = hydrateOnInteraction(
    resolveComponent('ComponentA'),
    hydrationTriggered
  );

  // wrap an async component
  const LazilyHydratedAsyncComp = hydrateOnInteraction(
    defineAsyncComponent(() => import('./ComponentB.vue')),
    hydrationTriggered
  );

  function triggerHydration() => {
    hydrationTriggered.value = true
  }
</script>

<template>
  <button @click="triggerHydration">Trigger</button>

  <LazilyHydratedComp />

  <LazilyHydratedAsyncComp />
</template>
```

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update unit tests as appropriate.

### Development

1. Clone the repository

   ```bash
   git clone https://github.com/freddy38510/vue3-lazy-hydration.git

   cd vue3-lazy-hydration
   ```

2. Install dependencies

   ```bash
   # with yarn
   yarn

   # with npm
   npm install
   ```

3. Start the development server which hosts a demo application to help develop the library

   ```bash
   # with yarn
   yarn dev

   # with npm
   npm run dev
   ```

## Credits

Many thanks to **Markus Oberlehner**, the author of the package
[vue-lazy-hydration](https://github.com/maoberlehner/vue-lazy-hydration).

## License

[MIT](https://github.com/freddy38510/vue3-lazy-hydration/blob/master/LICENSE)
