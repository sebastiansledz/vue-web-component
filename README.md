# Vue web component

> This is a stand-alone tutorial and can be used as a starting point for every next project that requires embedable widget to be placed on websites.

To make Vue.js application embedable as a widget anywhere using only one **app.js** and one **app.css** file we created a custom web component. We are using [vue-custom-element](https://github.com/karol-f/vue-custom-element), which allows us to render our Vue.js application as an HTML custom element like `<my-app></my-app>`.

**Hint:** there is an official [Vue web component wrapper](https://github.com/vuejs/vue-web-component-wrapper) available, however I found **vue-custom-element** as a project with more live development conducted by its contributors.

**Other dependencies and libraries used:**

* [document-register-element](https://github.com/WebReflection/document-register-element) - a stand-alone working lightweight version of the W3C Custom Elements specification, a custom polyfill in order for our HTML custom element to work.

## Create a project

Use Vue CLI command `vue create vue-custom-widget` or run `vue ui` in terminal to start GUI in the browser and create a project there. You will be prompted to pick a preset. You can either choose the default preset which comes with a basic Babel + ESLint setup, or select "Manually select features" to pick the features you need.

> For more information about how to create a project using Vue CLI please refer to [documentation](https://cli.vuejs.org/guide/creating-a-project.html#vue-create). To start this project we are using basic set of features, please see `package.json` of base vue-web-component repository for details.

## Optimize our application as an HTML custom element

This is what **vue-custom-element** is for. It’s a wrapper for our Vue.js application and allows us to invoke it in the source code via custom HTML element.

run ```npm install vue-custom-element --save```

rewrite initializing file `src/main.js` to have it as below

```javascript
// src/main.js

import Vue from 'vue'
import App from './App.vue'
import router from './router'
import vueCustomElement from 'vue-custom-element'

Vue.use(vueCustomElement)

App.router = router
Vue.customElement('vue-widget', App)
```

We are not using `new Vue()` anymore. Calling `Vue.customElement()` is doing it for us — and we provide the widget name (`<vue-widget></vue-widget>` in HTML) in the first argument and our App component in the second argument.

**Warning:** Using a name without a dash symbol may not work.

Anything you’d like to include in `new Vue()` before, is supposed to be included in `App` as a property now, calling `Vue.customElement()` after all your setup is done.

## Change our index.html

`public/index.html`

We remove our `<div id="app"></div>` and replace it with the new `<vue-widget></vue-widget>`.

**Caution:** For old(er) browsers we need a custom polyfill in order for our HTML custom element to work. Download it and import it in our main.js file.

run `npm install document-register-element --save`

```javascript
// src/main.js

import 'document-register-element/build/document-register-element'
```

## Run the build

`npm run build`

Building gives desired source — compressed, neat and small in size. But in case we are developing a larger app, we may experience having dozens of .js chunk files in our dist (build) folder. We wanted a single app.js file that could be imported in other sites and not fourty-one 1–3 kB .js files.

## Disable chunks

Create a **vue.config.js** file in the root of our application (next to the package.json). This is where webpack config is written in Vue CLI 3.

We will need to install webpack to use its **LimitChunkCountPlugin**. We will also need to pass the config to delete split chunks.

run `npm install webpack --save`

```javascript
// vue.config.js

const webpack = require('webpack')
module.exports = {
  configureWebpack: {
    plugins: [
      new webpack.optimize.LimitChunkCountPlugin({
        maxChunks: 1
      })
    ]
  },
  filenameHashing: false,
  chainWebpack:
    config => {
      config.optimization.delete('splitChunks');
    }
}
```

## Run build

`npm run build`

The build should contain only single **app.js** and single **app.css** file.

## Host the source

Put whole build source - everything in the **/dist** folder - to root of desired domain, example used: `https://vuiget-source.sledz.xyz`

If we are using fonts or images, they are included in the build folder **(/dist)** and bound in the style and script files - **app.js** and **app.css** files are seeking those assets.

To make them accessible for external sources we may need to add header to our domain configuration like .htaacess as below:

```
Header add Access-Control-Allow-Origin "*"
```

## Embed the widget

When all the needed assets are uploaded to source domain include **app.js** and **app.css** file in the site where you want to display the widget. Please see example below.

```html
<!doctype html>
<html>
<head>
 <!-- widget source css -->
 <link href="https://vuiget-source.sledz.xyz/css/app.css" rel="stylesheet">
</head>
<body>
 <vue-widget title="Vue Web Component"></vue-widget>
 <!-- widget source js -->
 <script src="https://vuiget-source.sledz.xyz/js/app.js"></script>
</body>
</html>
```

Please see the working demo here: [https://vuiget-demo.sledz.xyz](https://vuiget-demo.sledz.xyz)
