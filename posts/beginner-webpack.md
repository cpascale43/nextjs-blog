---
title: "A very beginner's guide to Webpack"
date: "2020-06-08"
---

In the Node environment, we have a "CommonJS" module system that uses module.exports/require to isolate parts of each file (or "module"). Up until ES6, there were no built-in "modules" in browser code.\* By default, each script in an HTML document is executed in order and shares one scope.

Enter...Webpack!
![Webpack logo. A dark blue and light blue diamond shape.](https://dev-to-uploads.s3.amazonaws.com/i/wh4z8ldndx0dnh7shbb1.png)

From the [Webpack 5 docs](https://webpack.js.org/concepts/):

> Webpack is a bundler for modules. The main purpose is to bundle JavaScript files for usage in a browser, yet it is also capable of transforming, bundling, or packaging just about any resource or asset.

What does this mean? Let's see Webpack in action by building a small JavaScript program in Node.

## Setup

Make a new project with npm and install `webpack` and `webpack-cli`.

```
mkdir hello-webpack && cd hello-webpack
npm init -y
npm install --save-dev webpack webpack-cli
```

Now, within your root folder, make the directories `src` and `public`. The `src` folder will hold our unprocessed source code, and we'll direct Webpack to output our transpiled code in the `public` folder. You'll also need to create a file called `webpack.config.js` - more on that later. Your project should look like this:

```
hello-webpack/
├── src/
├── public/
├── webpack.config.js
└── package.json
```

#### package.json

```json
{
  "name": "hello-webpack",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11"
  }
}
```

#### public/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <script src="../src/game.js" defer></script>
    <script src="../src/main.js" defer></script>
    <link rel="stylesheet" href="style.css" />
    <title>Click Me</title>
  </head>
  <body>
    <button id="button">Click Me!</button>
  </body>
</html>
```

#### public/style.css

```css
button {
  height: 300px;
  width: 300px;
  font-size: 40px;
  background-color: goldenrod;
  color: white;
  border-radius: 50%;
  cursor: pointer;
}
```

#### src/game.js

```javascript
let numTimesClicked = 0;

function win() {
  alert("You win!");
  reset();
}

function reset() {
  numTimesClicked = 0;
}

function click() {
  numTimesClicked++;
  console.log(`You've been clicked!`);
  if (numTimesClicked === 10) win();
}
```

#### src/main.js

```javascript
const button = document.getElementById("button");

button.addEventListener("click", function () {
  click();
});
```

## Why do you need Webpack?

From your command line, run `open public/index.html`. You should see a yellow button. When clicked, the button should log a message to your console. If you click the button 10 times, an alert should pop up letting you know - you've won! Great! We're done!

Just kidding. Take a look at your `index.html` file. What happens if you don't include the defer keyword in lines 7 and 8? What about if you re-order your JavaScript files?

```html
<!-- remove 'defer' from lines 7 and 8 -->
<!-- re-order 'game.js' and 'main.js' -->
<script src="../src/main.js"></script>
<script src="../src/game.js"></script>
```

Did you see something like this in your console?
![An error that says Uncaught TypeError: cannot read property addEventListener of null](https://dev-to-uploads.s3.amazonaws.com/i/il5ycz4xtc106xcwwklq.png)
Uh-oh.\*\* Remember that thing I said in the beginning about scripts executing in order? The `defer` attribute tells your browser not to run a specific JavaScript file until after the HTML file is done loading. Without `defer`, your JavaScript executes as soon as the HTML loads. And if the code in your 'main.js' file runs before the code in 'game.js', your program will try to run your 'click()' function before it's been defined.

Which is why you now have an error in your console.

## Bundling modules with Webpack

Now that we know why we need Webpack, let's see it in action.

Webpack is a module bundler. Its purpose is to process your application by tracking down its dependencies, then bundle them all up into one or more files that can be run in the browser. <b>Just like Node apps are universally configured by a `package.json`, you'll configure Webpack in your `webpack.config.js` file.</b>

### webpack.config.js

Webpack is based around several key components: an entry point, an output location, [loaders](https://webpack.js.org/concepts/loaders/), and [plugins](https://webpack.js.org/concepts/plugins/). I'll only focus on entry and output, but you'll definitely use the other two when you configure Webpack for larger projects.

#### Entry: The JavaScript file where Webpack begins building.

```javascript
module.exports = {
  entry: "./path/to/my/entry/file.js",
};
```

#### Output: Name and path for the bundled JavaScript.

```javascript
const path = require("path");

module.exports = {
  entry: "./path/to/my/entry/file.js", // the starting point for our program
  output: {
    path: path.resolve(__dirname, "directory_name"), // the absolute path for the directory where we want the output to be placed
    filename: "my-first-webpack.bundle.js", // the name of the file that will contain our output - we could name this whatever we want, but bundle.js is typical
  },
};
```

Your `webpack.config.js` file may look something like this:

```javascript
const path = require("path");

module.exports = {
  mode: "development", // could be "production" as well
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "public"),
    filename: "bundle.js",
  },
};
```

### NPM scripts

Now that we have our Webpack configuration, we need to add an npm script to our package.json. We can pick any word we want, but "build" is conventional. We can simply use "webpack." If we want Webpack to watch for changes and hot reload files, we can add a "--w" flag at the end. (If we didn't do this step, we would have to run a local copy of Webpack from the command line every time we wanted to run it.)

Your NPM scripts should look like this:

```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --w"
  },
```

Now... go ahead and fire her up!

Huh? What's this in my console?
![The webpack build log in your console. It shows the size of your bundle, the time you bundled it and your file names.](https://dev-to-uploads.s3.amazonaws.com/i/awg2dd6a6re48vjw4l6k.png)

That's your first bundle. The metadata in your console tells you how big your bundle is. Wow! Now that you've done this, you can use ES Modules. This means that as your program gets bigger, you can import and export functions between JavaScript files. Cool!

## Bring it to the web

We're almost done. We've configured Webpack to bundle our 'main.js' file and output a 'bundle.js' file in our /public directory.

Now, we can make use of ES Modules in our JavaScript. Remember how the `click` function was being invoked before it existed to the browser? Now, we can use `export` and `import` syntax to export it from `game.js` and call it within `main.js`, avoiding this problem altogether. Like this:

#### game.js

```
// below the click() function
export default click;
```

#### main.js

```
// at the top of main.js
import click from './game'
```

Lastly, we need to make a small change to our HTML file. Before we knew about Webpack, `index.html` loaded two separate JavaScript files. Now, all of the code in those files has been packaged into `bundle.js` - so we can simply point our script tag to `bundle.js`.

Go ahead and replace your script tags with a reference to `bundle.js`:

```html
<!-- <script src="../src/game.js" defer></script>
  <script src="../src/main.js" defer></script> -->
<script src="bundle.js" defer></script>
```

Now, run `open public/index.html`.

Does your program look and function exactly the same as before? Great! You've done everything right.

Take a peek in your DevTools, and navigate over to the 'Sources' tab. You should be able to click on `bundle.js` and observe your beautifully bundled JavaScript. Neat!

![Your JavaScript code has been bundled into one file. By navigating to the Sources tab, you can see the bundle.js file where your bundled code lives.](https://dev-to-uploads.s3.amazonaws.com/i/rlpwg46z64ts78myxp72.png)

## What did we learn?

Webpack is a bundling tool which packages up all of your JavaScript files into one neat file. We learned:

- Webpack bundles your JS code & helps support ES Modules
- Two main concepts are entry and output
- How to set up webpack.config.js

Great job! You've learned so much, and yet, there is still so much more to learn. From here, you may want to read about a compiler called [Babel](https://github.com/babel/babel-loader). Webpack is commonly used with Babel to transpile the latest JavaScript syntax across older browsers. You can also read about how Webpack handles CSS files, code splitting, and other fun things. It's also not the only tool of its kind - you could take a look at [grunt](https://gruntjs.com/), [gulp](https://gulpjs.com/), or [browserify](http://browserify.org/).

## Happy coding!👋

![Two side-by-side images of a man coding and a dog coding. The man thinks he is a god because he has learned everything about programming. On the other hand, the dog says, "I have no idea what I'm doing."](https://dev-to-uploads.s3.amazonaws.com/i/9adjjkca4j798czpzr6r.png)

### \*The "import" and "export" keywords were introduced in ES6 and there is native browser support for these, though it's not yet universal. To natively load ES modules, you can specify the `type="module"` attribute on your script tags. However, this would result in as many HTTP requests as there are JavaScript files in your HTML file. As your applications grow, you won't want to deal with that, so it's still a good idea to know about bundlers and transpilers.

### \*\*From [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script): 'This Boolean attribute is set to indicate to a browser that the script is meant to be executed after the document has been parsed, but before firing DOMContentLoaded.'
