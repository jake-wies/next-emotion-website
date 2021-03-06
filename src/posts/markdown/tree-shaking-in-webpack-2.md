---
title: Tree-Shaking ES6 Modules in webpack 2
date: 01-16-2017
---

Webpack 2 is still in its beta stage at the time of writing this, but should see its release very soon. It brings with it a variety of anticipated features. One of those features includes native support for ES6 modules.

#### So what? 

So instead of using the `var module = require('module')` syntax, webpack 2 supports ES6 `imports` and `exports`. This proves quite powerful, and opens the door for some code optimizations such as **tree-shaking**.

## What is tree-shaking?

Popularized by Rich Harris' [Rollup.js](http://rollupjs.org/) module bundler, *tree-shaking* is the ability to only include code in your bundle that is being *used.* When I first played around with Rollup, I was amazed at how well it worked with ES6 modules. The development experience just felt...right. I can create separate modules written in "future JavaScript", and then include them anywhere in my code. Any code that goes unused doesn't make it into my bundle. Genius! 

#### What problem does it solve? 

If you're writing JavaScript in 2017 and *understand* (see: [JavaScript fatigue](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f#.nk6chuvta)) the various tools around, your development experience probably feels pretty fluid. This is important, but what's also important is *user experience*. A lot of this modern tooling ends up bloating web applications with massive JavaScript files, resulting in performance loss.

What I love about Harris' Rollup is that it takes a stab at this issue and brings a solution to the forefront of the JavaScript community. Now big names like webpack are attempting to iterate on it. 

#### A simple example

Before we get started I want to provide you with a trivial example of tree-shaking. Our application is made up of 2 files, `index.js` and `module.js`. 

Inside of `module.js` we export 2 named arrow functions:

```
// module.js
export const sayHello = name => `Hello ${name}!`;

export const sayBye = name => `Bye ${name}!`;
```

Only `sayHello` is imported into `index.js` file:

```
// index.js
import { sayHello } from './module';

sayHello('World');
```

`sayBye` is exported but *never* imported. Anywhere. Therefore, due to tree-shaking, it won't be included in our bundle:

```
// bundle.js
const sayHello = name => `Hello ${name}!`;

sayHello('World');
```

Depending on the bundler used, the output file above may look different. It's just a simplified version, but you get the idea!

Recently I read an article written by [Roman Liutikov](https://medium.com/@roman01la/dead-code-elimination-and-tree-shaking-in-javascript-build-systems-fb8512c86edf#.69aadkgrb), and he made a great analogy in order to visualize the concept of tree-shaking:

> If you wonder why it’s called tree-shaking: think of your application as a dependency graph, this is a tree, and each export is a branch. So if you shake the tree, the dead branches will fall. 

## Tree-shaking in webpack 2

Unfortunately for those of us using webpack, tree-shaking is "behind a switch", if you will. Unlike Rollup, some configuration needs to be done before we can get the functionality we're looking for. The "behind a switch" part might confuse some people. I'll explain.

#### Step 1: Project setup

I'm going to assume that you understand webpack basics and can find your way around a basic webpack configuration file. Let's start by creating a new directory:

```
mkdir webpack-tree-shaking && cd webpack-tree-shaking
```

Once inside, let's initialize a new `npm` project:

```
npm init -y
```

The `-y` option generates the `package.json` quickly without requiring you to answer a bunch of questions. 

Next, let's install a few project dependencies:

```
npm i --save-dev webpack@beta html-webpack-plugin
```

The command above will install the latest beta version of webpack 2 locally in our project as well as a useful plugin named `html-webpack-plugin`. The latter is not necessary for the goal of this walkthrough but will make things a bit quicker.

Open up `package.json` and make sure they've been installed as `devDependencies`.

#### Step 2: Create JS files

In order to see tree-shaking in action we need to have some JavaScript to play around with. In your project's root, create a `src` folder with 2 files inside:

```
mkdir src && cd src

touch index.js

touch module.js

```

Copy the code below into the correct files:

```
// module.js
export const sayHello = name => `Hello ${name}!`;

export const sayBye = name => `Bye ${name}!`;
```

```
// index.js
import { sayHello } from './module';

const element = document.createElement('h1');

element.innerHTML = sayHello('World');

document.body.appendChild(element);
```

If you've gotten this far, your folder structure should look like this:

```
/
| - node_modules/
| - src/
|   | - index.js
|   | - module.js
| - package.json
```

#### Step 3: Webpack from the CLI

Since we have no configuration file created for our project, the only way to get webpack to do any work at the moment is through the webpack CLI. Let's perform a quick test. 

In your terminal, run the following command in your project's root:

```
node_modules/.bin/webpack
```

After running this command, you should see output like this:

```
No configuration file found and no output filename configured via CLI option. 
A configuration file could be named 'webpack.config.js' in the current directory. 
Use --help to display the CLI options.
```

The command doesn't do anything, and the webpack CLI confirms this. We haven't given webpack any information about what files we want to bundle up. We could provide this information via the command line *or* a configuration file. Let's choose the former just to test that everything is working:

```
node_modules/.bin/webpack src/index.js dist/bundle.js
```

What we've done now is pass webpack an `entry` file and an `output` file via the CLI. This information tells webpack, "go to `src/index.js` and bundle up all the necessary code into `dist/bundle.js`". And it does just that. You'll notice that you now have a `dist` directory containing `bundle.js`.

Open it up and check it out. There's some extra javascript in the bundle necessary for webpack to do its thing, but at the bottom of the file you should see your own code as well. 

#### Step 4: Create a webpack configuration file

Webpack can handle a lot of things. I've spent a good chunk of my free time diving into this bundler and I still have barely scratched the surface. Once you've move passed trivial examples its best to leave the CLI behind and create a configuration file to handle the heavy lifting. 

In your project's root, create a `webpack.config.js` file:

```
touch webpack.config.js
```

This file can be as complicated as you make it. We'll keep it light for the sake of this post:

```
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: 'dist'
  },
  plugins: [
    new HtmlWebpackPlugin({ title: 'Tree-shaking' })
  ]
}
```

This file provides webpack with the same information we gave to the CLI earlier. We've defined `index.js` as our `entry` file and `bundle.js` as our `output` file. We've also added our `html-webpack-plugin` which will generate an html file in our `dist` directory. Convenient.

Go ahead and test this to make sure it's still working. Remove your `dist` directory, and in the command line type:

```
webpack
```

If everything went smoothly, you can open up `dist/index.html` and see "Hello World!".

==**Note:** The use of a configuration file gives us the convenience of typing `webpack` instead of `node_modules/.bin/webpack`. Small wins.==

#### Step 5: Babel

I mentioned earlier that webpack 2 brings native support for ES6 modules. This is all true, but it doesn't change the fact that ES6 is not fully supported across all browsers. Because of this, we're required to *transform* our ES6 code into readily acceptable JavaScript using a tool like [Babel](http://babeljs.io/). In conjunction with webpack, Babel gives us the ability to write our "future JavaScript" without worrying about the implications of unsupported browsers.

Let's go ahead and install Babel in our project:

```
npm i --save-dev babel-core babel-loader babel-preset-es2015
```

Take note of the `babel-preset-es2015` package. This little guy is the reason I sat down to write all of this up.

#### Step 6: `babel-loader`

Webpack can be configured to transform specific files into modules via [loaders](https://webpack.js.org/concepts/#loaders). Once they are transformed, they are added to a dependency graph. Webpack uses the graph to resolve dependencies and includes only what is needed into the final bundle. This is the basis for how webpack works. 

We can now configure webpack to use `babel-loader` to transform all of our `.js` files:

```
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: { filename: 'bundle.js', path: 'dist' },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: { 
          presets: [ 
            'es2015' 
          ] 
        }
      }
    ]
  },
  plugins: [ 
    new HtmlWebpackPlugin({ title: 'Tree-shaking' }) 
  ]
};
```

The `module` property provides a set of instructions for webpack. It says, "take any files ending in `.js` and transform them using `babel-loader`, but don't transform any files inside of `node_modules`!" 

We're also passing the `babel-preset-es2015` package as an option to `babel-loader`. This just tells `babel-loader` *how* to transform the JavaScript.

Run `webpack` again to make sure everything is good. Yes? Great! What we've done is bundled up our JavaScript files while compiling them down to JavaScript thats readily supported across browsers. 


## The underlying problem

The package `babel-preset-es2015` contains another package named `babel-plugin-transform-es2015-modules-commonjs` that turns all of our ES6 modules into `CommonJS` modules. This isn't ideal, and here's why.

Javascript bundlers such as webpack and Rollup can only perform tree-shaking on modules that have a static structure. If a module is static, then the bundler can determine its structure at build time, safely removing code that isn't being imported anywhere. 

`CommonJS` modules do not have a static structure. Because of this, webpack won’t be able to tree-shake unused code from the final bundle. Luckily, Babel has alleviated this issue by providing developers with an option that we can pass to our `presets` array along with `babel-preset-es2015`:

```
options: { 
  presets: [ 
    [ 'es2015', { modules: false } ] 
  ]         
}
```

According to Babel's [documentation](https://github.com/babel/babel/tree/master/packages/babel-preset-es2015#options): 

*"`modules` - Enable transformation of ES6 module syntax to another module type (Enabled by default to "commonjs").
Can be `false` to not transform modules, or one of `["amd", "umd", "systemjs", "commonjs"]`"*.

Slide that extra bit of code into your configuration and you'll be cooking with peanut oil.

The final state of `webpack.config.js` looks like this:

```
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: { filename: 'bundle.js', path: 'dist' },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: { 
          presets: [ 
            [ 'es2015', { modules: false } ] 
          ] 
        }
      }
    ]
  },
  plugins: [ new HtmlWebpackPlugin({ title: 'Tree-shaking' }) ]
};

```

## The Grand Finale
Run `webpack` again and pop open your `bundle.js` file. You won't notice any difference. Before you go crazy, know this! It's ok. We've been running webpack in development mode this whole time. Webpack knows that we have unused exports in our code. Even though it's included in the final bundle, `sayBye` will never make it to production. 

If you still don't believe me, run `webpack -p` in your terminal. The `-p` option stands for *production*. Webpack will perform a few extra performance optimizations, including minification, removing any unused code along the way. 

Open up `bundle.js`. Since it's minified, go ahead and search for `Hello`. It *should* be there. Search for `Bye`. It *shouldn't*. 

Voila! We now have a working implementation of tree-shaking in webpack 2!

For the curious, I've been slowly iterating over my own lightweight webpack configuration in a [GitHub Repo](https://github.com/jake-wies/webpack-hotplate). It's not meant to be overly verbose and bloated. It's focused on being an approachable boilerplate with walkthroughs at every turn. If you're interested, check it out!