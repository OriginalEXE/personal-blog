---
title: Server rendering emotion v10 in Nuxt
date: "2018-12-09T18:59:50.000Z"
description: "How to use emotion v10 and Nuxt together"
---

Just when [I thought I had it figured out](https://antesepic.com/server-rendering/), emotion v10 comes out and breaks my server rendering...

A few more hours of debugging and I fond a solution (though it's more hacky than the previous one).

We no longer need `emotion-server` package, instead, we want to `npm install create-emotion-server --save`. Then, in each Nuxt layout file, import the emotion cache like this: `import { css, cache } from 'emotion';` (you can leave out css if you are not using it in the file). Here comes the hack part: `global.emotionCache = cache;` (add it somewhere below your import)

😱😱😱

That's right, we are using globals. We need to do this in order to later reference that cache inside the nuxt.config.js file. So let's open that file now, and import the `createEmotionServer` helper

    const createEmotionServer = require ('create-emotion-server').default;

We now just need to hook into a nuxt hook that will allows us to modify the HTML response from the server.

Here is what those might look like (added to the object you are exporting from your `nuxt.config.js` file):

```js
    render: {
      route (url, result) {
        const { renderStylesToString } = createEmotionServer (global.emotionCache);
        const withCss = renderStylesToString (result.html);

        result.html = withCss;
      },
    },
    generate: {
      page (page) {
        const { renderStylesToString } = createEmotionServer (global.emotionCache);
        const withCss = renderStylesToString (page.html);

        page.html = withCss;
      },
    },
```

I am using two different hooks because I am statically exporting my website, and a render hook is not called with `nuxt generate`.

Aaand the server side rendering works properly again :) See you on the next update!
