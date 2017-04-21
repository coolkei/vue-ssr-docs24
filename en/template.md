# Using a Page Template

When you render a Vue app, the renderer only generates the markup of the app. We also need to wrap the output with an extra HTML page shell.

You can directly provide a page template when creating the renderer. Most of the time we will put the page template in its own file, e.g. `index.template.html`:

``` html
<!DOCTYPE html>
<html lang="en">
  <head><title>Hello</title></head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

Notice the `<!--vue-ssr-outlet-->` comment -- this is where your app's markup will be injected.

We can then read and pass the file to the Vue renderer:

``` js
const fs = require('fs')
const Vue = require('vue')
const { createRenderer } = require('vue-server-renderer')

const renderer = createRenderer({
  template: fs.readFileSync('./index.template.html', 'utf-8')
})

const app = new Vue({
  template: '<div>hello</div>'
})

renderer.renderToString(app, (err, html) => {
  console.log(html) // will be the full page with app content injected.
})
```

## Interpolation Using the Render Context

> requires version 2.3.0+

The template supports simple interpolation, using double-mustache `{{ }}` for HTML-escaped content and triple-mustache `{{{ }}}` for non-escaped content. The interpolation is done using an optional "render context" object passed to `renderToString` as the 2nd argument:

``` js
const renderer = createRenderer({
  template: `
    <html>
      <head>
        <title>{{ title }}</title>
        {{{ meta }}}
      </head>
      <body>
        <!--vue-ssr-outlet-->
      </body>
    </html>
  `
})

const context = {
  title: 'hello',
  meta: `
    <meta ...>
    <meta ...>
  `
}

renderer.renderToString(app, context, (err, html) => {
  // page title will be "hello"
  // with meta tags injected
})
```

This might seem similar to simply interpolating with ES2015 template strings, but the `template` option can provide a few extra advantages (which will be covered later in the guide):

- The render context object can be passed to our Vue app, so that components can dynamically attach data to the context. This opens up possibilities such as component-based head management and critical CSS injection.

- When using the [BundleRenderer](./bundle-renderer.md) with enough build information, it's possible for the renderer to automatically infer and inject the proper asset/script links into the template.
