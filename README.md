# 🔗 Very Simple Links

A very simple turbolinks inspired library. You can use it for progressive
enhancements and to give your multi-page websites a SPA-like feel.

💾 only ~1kb (minify and gzip)

## Installation

```
npm install @very-simple/links
```

## Usage

Intercept link clicks and let very simple links take care of fetching the new
page, merging its `<head>` and swapping in its `<body>`.

```js
// shared.js
import links from '@very-simple/links'

const initPage = () =>
  document.querySelectorAll('a').forEach((el) =>
    el.addEventListener('click', async (event) => {
      event.preventDefault()
      links.visit(el.getAttribute('href'))
    })
  )

document.addEventListener('DOMContentLoaded', initPage)
links.on('visit', initPage)

links.start()
```

```html
<!-- page-a.html -->
<html>
  <head>
    <!-- Shared scripts in the `<head>` get only executed once. -->
    <script src="./shared.js" type="module"></script>
    <title>Page A</title>
  </head>
  <body>
    Content A
    <a href="page-b.html">Go to Page B</a>
    <script>
      // Scripts in the body get executed on each page visit.
      console.log('hello from Page A!')
    </script>
  </body>
</html>
```

```html
<!-- page-b.html -->
<html>
  <head>
    <script src="./shared.js" type="module"></script>
    <title>Page B</title>
  </head>
  <body>
    Content B
    <a href="page-b.html">Go to Page A</a>
  </body>
</html>
```

Checkout the `/examples` folder for more examples.

## Documentation

### Start

```js
links.start()

// In some rare cases you might not want to watch popstate navigation. You can
// disable it by calling `links.start({ watchHistory: false })`.
```

### Visit

```ts
links.visit('some/url/', options?)
```

```ts
interface VisitOptions {
  // Which `window.history` action should be performed, default is 'push'.
  action?: 'push' | 'replace' | 'none'

  // Wether or not to load the page from cache. Default is false (this will
  // still use a cached version for popstate events to simulate the default
  // browser behavior).
  useCache?: boolean

  // Only needed in rare cases. Provide a custom id under which the current page
  // will be cached before visiting the new URL.
  cacheId?: string
}
```

## Custom Container

By default the body is swapped, but you can specify a custom container instead.
This is useful for partial page refreshes.

```html
<meta name="simple-container" content="#my-container" />
```

See `/examples/partial` for a full example.

## Permanent Elements

If two pages contain the same element, it can be useful to keep the element
alive instead of replacing it. For example, an image that shouldn't flicker or
an animated header that shouldn't be reset between page visits.

Note: Permanent elements must have an id so they can be identified in both pages.

```html
<img data-simple-permanent id="my-permanent-image" src="img.gif" />
```

See `/examples/partial`, which also includes a permanent image.

## Progress Bar

Very Simple Links doesn't render a progress bar but provides you with everything
to easily implement your own.

Note: you have to include the progress bar element in every page.

```html
<head>
  <link
    rel="stylesheet"
    href="node_modules/very-simple/links/dist/progress.css"
  />
</head>
<body>
  <!-- Make the progressbar permanent so the css transitions work even when
  the new content is swapped in. -->
  <div id="progress" data-simple-permanent></div>
</body>
```

```css
#progress {
  position: fixed;
  top: 0;
  left: 0;
  /* The progress is available as a CSS variable (in %) */
  width: var(--simple-progress);
  height: 3px;
  opacity: 0;
  background-color: blue;
  /* Animate the progress and make sure the animation can finish by delaying
  the opacity */
  transition: width 300ms ease-out, opacity 300ms steps(1);
}

/* A data attribute is added to the document root to indicate loading */
:root[data-simple-loading] #progress {
  opacity: 1;
}
```

## Together with Very Simple Router

Using very simple links together with very simple router opens up a lot of
possibilities. You can use multiple html files that can each have their own
virtual routes.

```js
import links from '@very-simple/links'
import router from '@very-simple/router'
import { useRouter } from '@very-simple/links/useRouter'

// Connect links and router.
useRouter(links, router)
router.start()
// Make sure to disable watch history, as the router already does this!
links.start({ watchHistory: false })
```

See `examples/router` for a complete example.
