# Example typescript library setup

This is an example to setup a project for developing a library in typescript, with
the following features:

The sample package
- is developed in typescript
- should be usable in a browser, served locally without web-server (i.e. through the file:/// protocol)
- should be usable from Node.js
- should be compatible with the different existing Javascript module systems, in particular CommonJS, Asynchronous Module Definition (AMD), Universal Module Definition (UMD) and ES6 Modules (ESM)



## Creating an account on https://www.npmjs.com

In the end, the result should be published on npmjs as a scoped package, so we start with creating an account there, and an organization dedicated to the project: The organization name I'll pick is:

- typescript-webpack-library-setup



## Setup a new Node.js project locally

- Make sure Node.js is installed (from https://nodejs.org)
- Create an empty folder typescript-webpack-library-setup
- Within that folder, run `npm init --scope=@typescript-webpack-library-setup`

```bash
$ npm init --scope=@typescript-webpack-library-setup
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (@typescript-webpack-library-setup/typescript-webpack-library-setup)
version: (1.0.0)
description: A sample typescript library setup, bundled to be used locally within a browser and through Node.js
entry point: (index.js) dist/index.js
test command:
git repository: https://github.com/bene51/typescript-webpack-library-setup
keywords: typescript, webpack, library
author: Benjamin Schmid
license: (ISC)
About to write to D:\typescript-webpack-library-setup\package.json:

{
  "name": "@typescript-webpack-library-setup/typescript-webpack-library-setup",
  "version": "1.0.0",
  "description": "A sample typescript library setup, bundled to be used locally within a browser and through Node.js",
  "main": "dist/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/bene51/typescript-webpack-library-setup.git"
  },
  "keywords": [
    "typescript",
    "webpack",
    "library"
  ],
  "author": "Benjamin Schmid",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/bene51/typescript-webpack-library-setup/issues"
  },
  "homepage": "https://github.com/bene51/typescript-webpack-library-setup#readme"
}


Is this OK? (yes)
```



## Setup typescript

- Install typescript locally via `npm install --save-dev typescript ts-loader`
- Create a default typescript configuration file via `npx tsc --init`
- Modify `tsconfig.json`:
  - `"target": "es2016"`: the target js version
  - `"lib": ["ESNext", "dom"]`: the libraries available within typescript
  - `"module": "commonjs"`: the js module system to use, seems not to matter because webpack will change it
  - `"moduleResolution"`: "node10"
  - `"outDir": "./dist/"`: put the typescript transpiler output in the `dist` folder
  - `"rootDir": "./src/"`: creates the same folder hierarchy in the output folder as it exists under `src`
  - `"include": ["src"]`: transpile everything within the `src` folder
  - `"sourceMap": true`: keep the typescript sources, for debugging, e.g. in browsers
  - `"declaration": true`: keep ts types after transpiling to js, in a .d.ts file
  - `"alwaysStrict": true`
  - `"noUnusedParameters": true`
  - `"noImplicitReturns": true`
  - `"noImplicitAny": true`
  - `"strictNullChecks": true`
  
  Here is the final version:
```json
{
  "compilerOptions": {
    "target": "es2016",
    "lib": ["ESNext", "dom"],
    "module": "commonjs",
    "rootDir": "./src/",
    "moduleResolution": "node10",
    "baseUrl": "./",
    "declaration": true,
    "sourceMap": true,
    "outDir": "./dist/",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "alwaysStrict": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  },
  "include": ["src"]
}
```

- Modify `package.json`:
```json
"types": "dist/index.d.ts"
```



## Create the source

- Create source and output folders: `mkdir src/ dist/`
- Create `src/index.ts` with the following contents:
```typescript
export module Test {
  export function helloworld(): string {
    return "Hello world!";
  }
}
```



## Setup webpack to create a bundled js file in 'umd' format

- Install webpack via `npm install --save-dev webpack webpack-cli`
- Create `webpack.config.js` (https://webpack.js.org/guides/typescript/, https://webpack.js.org/guides/author-libraries/#authoring-a-library)
```javascript
const path = require('path');

module.exports = {
  module: 'production',
  entry: './src/index.ts',
  devtool: 'inline-source-map',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'index.js',
    library: {
      name: 'ts_wp_lib',
      type: 'umd'
    },
    globalObject: 'this',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

This creates a bundled javascript module in 'umd' format. The resulting js file is compatible with the Javascript AMD and CommonJS module system, and can also be used in browsers, even served locally without HTTP server. The next paragraph demonstrates an example.



## Build the project

```bash
npx webpack
```
This will create `dist/index.js` as well as `dist/index.d.ts`.



## Use the library in a local HTML file

- Create `index.html`:
```html
<html>
<body>
  <script src="dist/index.js"></script>
  <script>
    document.body.innerHTML = "<h1>" + ts_wp_lib.Test.helloworld()  + "</h1>";
  </script>
</body>
</html>
```

Open it in a browser, and you should see the greeting.




## Create a library compatible with the ES module system

The 'udm' module created above is compatible with browsers, and can be used in Node.js using the CommonJS module system (`require('...')`), but is not compatible with the ES module system. The ES module system, however, is currently state of the art and needs to be supported. To publish our package as ES module, the following steps need to be performed:

- Modify `package.json`:
```json
  "main": "./dist/index.cjs",
  "type": "module",
  "exports": {
    "import": "./dist/index.js",
    "require": "./dist/index.cjs"
  },
```

So our package type is set to `module`, which implies that any .js file will be expected to be ESM compatible. Files in CommonJS format should have the ending `.cjs`. So wee need to modify the name of the created UMD library to `index.cjs`.

Our package can then provide multiple versions of our library, depending in which context the library is loaded. If loaded using `require`, `dist/index.cjs` will be used, if loaded using `import`, `dist/index.js` will be used. This is what the `exports` entry achieves. We will test this later.

Since `webpack.config.js` is also in CommonJS format, we'll first change it to the ES module format and at the same time rename it to `webpack-umd.config.js`:

```javascript
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default {
  mode: 'production',
  entry: './src/index.ts',
  devtool: 'inline-source-map',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'index.cjs',
    library: {
      name: 'ts_wp_lib',
      type: 'umd'
    },
    globalObject: 'this',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

To create a second version compatible with ES modules, we use a second webpack configuration file, `webpack-esm.config.js:
```javascript
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default {
  mode: 'production',
  entry: './src/index.ts',
  devtool: 'inline-source-map',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'index.js',
    library: {
      type: 'module'
    },
    globalObject: 'this',
    path: path.resolve(__dirname, 'dist'),
  },
  experiments: {
    outputModule: true,
  },
};
```

Now we can build both of them with

```bash
npx webpack -c webpack-umd.config.js
npx webpack -c webpack-esm.config.js
```

Modify `index.html` to use the 'umd' file:
```html
  <script src="dist/index.cjs"></script>
```

And open it in a browser to verify that it still works.




## Test it with Node.js

- Create a temporary directory outside the project tree <tmp>
- Create a default package.json: `npm init -y` 
- Install our package: `npm install path/to/typescript-webpack-library-setup`
- Create a file `index-umd.cjs`:

```javascript
l = require('@typescript-webpack-library-setup/typescript-webpack-library-setup');
console.log(l.Test.helloworld());
```

- Create a file `index-esm.mjs`:

```javascript
import { Test } from '@typescript-webpack-library-setup/typescript-webpack-library-setup';
console.log(Test.helloworld());
```

- Test both via `node.js index.mjs` and `node.js index.cjs`.

_Note_: Since we did not put `"type": "module"` in `package.json`, Node.js expects *.js files to be in CommonJS format. We call the file using CommonJS syntax `index-umd.cjs`, and the file using ESM syntax `index-esm.mjs`, because `.cjs` files will always be assumed to be in CommonJS format, while `.mjs` files will always be assumed to be in ESM format.



## Put the code under version control and publish it on Github

- Add a file `.gitignore`:
```text
dist/
node_modules/
package-lock.json
```
- Create an empty git repository `git init`
- Add all files: `git add -A`
- Commit it: `git commit -m "Initial commit"`
- On https://github.com, create a new repository `typescript-webpack-library-setup.git`
- Locally:
  - `git branch -M main`
  - `git remote add origin https://github.com/bene51/typescript-webpack-library-setup.git`
  - `git push -u origin main`



## Publish it on npm.js

- Filter what's being published: The npm registry should only get the files needed for deployment, i.e. no configuration files, no source files, basically only the files under `dist/`, the `README.md`, and the `package.json`

- Some people also don't recommend to publish tests, while others do.

- By default, everything that's in .gitignore will be ignored by npm, too. But in our case, this is not what we want: Git ignores e.g. the dist/ folder, which is exactly what we need to put on npm.js.

- There are two ways to specify what should belong to a Node.js package. Either specify a whitelist in `package.json`, under the `files` entry, or specify a blacklist in .npmignore. We'll follow the former approach here, i.e. change `package.json`:
```json
  "files": ["README.md", "package.json", "./dist/"]
```

- Now check with `npm pack --dry-run` which files would get published, and adjust `files` if necessary.

- If everything is OK, the package can be published:
```bash
 npm login --scope=@typescript-webpack-library-setup --registry=https://registry.npmjs.com
npm publish --access public
```

