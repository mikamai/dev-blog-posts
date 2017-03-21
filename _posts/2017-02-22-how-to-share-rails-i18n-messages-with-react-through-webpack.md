---
ID: 1121
post_title: >
  How to share Rails i18n messages with
  React through webpack
author: Nicola Racco
post_date: 2017-02-22 11:51:44
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/02/22/how-to-share-rails-i18n-messages-with-react-through-webpack/
published: true
---
Very specific title hu? This time I'm here for sharing a solution for a specific problem I encountered a couple of days ago.

There's this existing Rails monolithic application. Team and customer decided that time had come for this app to be decoupled in two components: Rails would do its usual work as an administration and API backend, while React would be used for the frontend component. Everything related to the frontend would then be rewritten, keeping the same behaviour and visual design. But there are a lot of translations related to the user experience and that have now to be included in the javascript bundle, while they were before used by the server.

To do so, I was aiming to write a webpack plugin that could give access to the Rails yml translations. Then I discovered the [Virtual Module Plugin](https://github.com/rmarscher/virtual-module-webpack-plugin): During webpack compilation this plugin [injects a fake file](https://github.com/rmarscher/virtual-module-webpack-plugin/blob/master/index.js#L54) containing the given text.

Working on it led me to an extension that flattens and joins the various translations, providing them to javascript source files without duplicating data: a [Rails Translations Plugin](https://github.com/mikamai/rails-translations-webpack-plugin). In our case we didn't even want to share ALL translations between Rails and React, but just a `frontend.yml` file (that stores messages related to the user experience), so the resulting `webpack.config.js` is:

```js
// app/frontend/webpack.config.js

const clientApp = path.resolve(__dirname, &#039;app&#039;);
const webpackConfig = {
  entry: [path.join(clientApp, &#039;index.js&#039;)],
  output: {
    path: path.resolve(__dirname, &#039;dist&#039;),
    filename: &#039;bundle.js&#039;
  },
  plugins: [
    new RailsTranslationsPlugin({
      root       : clientApp,
      localesPath: path.resolve(__dirname, &#039;../backend/config/locales&#039;),
      pattern    : &#039;**/frontend.yml&#039;
    })
  ],
  module: {
    loaders: [
      {
        include : clientApp,
        test    : /\.(js|jsx|es6)$/,
        loader  : &#039;babel-loader&#039;
      },
      {
        include : clientApp,
        test    : /\.json$/,
        loader  : &#039;json-loader&#039;
      }
    ]
  }
};
```

And this allowed us to use them in [ReactIntl](https://github.com/yahoo/react-intl):

```js
// app/context.jsx
// ...
import { addLocaleData } from 'react-intl';
import translations from 'translations.json';

for (let locale of Object.keys(translations)) {
  addLocaleData(require(`react-intl/locale-data/${locale}`));
}

const locale   = navigator.language || 'en';
const messages = translations[locale];

return (
  <IntlProvider locale={locale} key={locale} messages={messages}>
    <App />
  </IntlProvider>
)
```

The only downside is that React Intl is not able, obviously, to work with Rails localization formats, and in fact we needed to iterate over the various locales and require its additional data needed for localization (look at the `addLocaleData` method in the above snippet). I'm now thinking of improving this plugin adding builtin methods that work like Rails's `I18n.t` and `I18n.l`.