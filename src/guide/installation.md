# Installation

## Release Notes

Latest beta version: 3.0.0-rc.1

Detailed release notes for each version are available on [GitHub](https://github.com/vuejs/vue-next/releases).

## Vue Devtools

> Vue Devtools for Vue 3 requires at least `vue@^3.0.0-rc.1`

When using Vue, we recommend also installing the [Vue Devtools](https://github.com/vuejs/vue-devtools#vue-devtools) in your browser, allowing you to inspect and debug your Vue applications in a more user-friendly interface.

[Get the Chrome Extension](https://chrome.google.com/webstore/detail/vuejs-devtools/ljjemllljcmogpfapbkkighbhhppjdbg)

[Get the Firefox Addon](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)

[Get the standalone Electron app](https://github.com/vuejs/vue-devtools/blob/dev/packages/shell-electron/README.md)

## CDN

For prototyping or learning purposes, you can use the latest version with:

```html
<script src="https://unpkg.com/vue@next"></script>
```

For production, we recommend linking to a specific version number and build to avoid unexpected breakage from newer versions.

## NPM

NPM is the recommended installation method when building large scale applications with Vue. It pairs nicely with module bundlers such as [Webpack](https://webpack.js.org/) or [Browserify](http://browserify.org/). Vue also provides accompanying tools for authoring [Single File Components](../guide/single-file-component.html).

```bash
# latest stable
$ npm install vue@next
```

## CLI

Vue provides an [official CLI](https://github.com/vuejs/vue-cli) for quickly scaffolding ambitious Single Page Applications. It provides batteries-included build setups for a modern frontend workflow. It takes only a few minutes to get up and running with hot-reload, lint-on-save, and production-ready builds. See [the Vue CLI docs](https://cli.vuejs.org) for more details.

::: tip
The CLI assumes prior knowledge of Node.js and the associated build tools. If you are new to Vue or front-end build tools, we strongly suggest going through <a href="./">the guide</a> without any build tools before using the CLI.
:::

For beta, Vue CLI now has experimental support via [vue-cli-plugin-vue-next](https://github.com/vuejs/vue-cli-plugin-vue-next).

## Vite

[Vite](https://github.com/vitejs/vite) is a web development build tool that allows for lighting fast serving of code due its native ES Module import approach. 

Vue projects can quickly be set up with Vite by running the following commands in your terminal.

With NPM:

```bash
$ npm init vite-app <project-name>
$ cd <project-name>
$ npm install
$ npm run dev
```

Or with Yarn:

```bash
$ yarn create vite-app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```
