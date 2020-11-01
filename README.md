
Stenciljs provides a way to generate compiled web components.
However, I have found it far from obvious how to construct a component
that is completely stand-alone; the instructions for 'building' assume
familiarity with the node.js workflow.

Here is a sequence of steps that I have found to work.

## get started with the starter component

```
git clone https://github.com/ionic-team/stencil-component-starter.git my-component
cd my-component
git remote rm origin
ls

LICENSE  package.json  readme.md  src/  stencil.config.ts  tsconfig.json
```

## rename stencil-starter-project-name
change this to 'my-component'.  This involves changing several files,
found as follows

```
grep 'stencil-starter' *

package.json:  "name": "stencil-starter-project-name",
package.json:  "unpkg": "dist/stencil-starter-project-name/stencil-starter-project-name.js",
stencil.config.ts:  namespace: 'stencil-starter-project-name',

grep -r 'stencil-starter' */*

src/index.html:    <script type="module" src="/build/stencil-starter-project-name.esm.js"></script>
src/index.html:    <script nomodule src="/build/stencil-starter-project-name.js"></script>


```

Manually change all of these instances to 'my-component'

## install node.js

The workflow requires node.js, which also provides copy of 'npm', the
node package manager.

## install dependencies
```
 npm install

 ls
```
This adds a 'node_modules' directory -- which is 16MB in size!


'npm' is the node package manager.

## start the app

```
npm start
```
This runs the 'start' script as specified in 'scripts' object of package.json.
```
"start": "stencil build --dev --watch --serve",
```
A side effect is that a dev server is started, and
the project is run (somehow) by this server.
Exactly what gets run is a mystery to me.

There is now a 'dist' folder. Presumably, the 'compiled' version of
'my-component' is somewhere in that dist folder.

```
ls dist

custom-elements/  my-component/  types/
```

```
ls dist/my-component

app-globals-0f993ce5.js  index.esm.js           my-component.esm.js
css-shim-6962f98b.js     index-3962282a.js      shadow-css-a637b891.js
dom-58af1012.js          my-component.entry.js


```


## 'build'

```
npm run build

# this adds some directories in dist
ls dist

cjs/         custom-elements/  index.cjs.js  my-component/
collection/  esm/              index.js      types/


# esm  is thought to be the 'standalone' code the component, using 
# es2015 syntax

ls dist/esm

index.js           loader.js              my-component.js
index-5dcbdb1a.js  my-component.entry.js  polyfills/

The biggest thing in this is the polyfills directory.
But I don't think it is required when using a modern browser.

```

## create a 'build' directory for standalone

```
cp -r dist/esm build
du -sh build
# 181k
rm -r build/polyfills
du -sh build
# 40k
```

## make an application that uses *build*

```
cp src/index.html index-build.html

# manually change 'src="/build' to 'src="build' in index-build.html
# This occurs twice. 
# We are using 'build/my-component.esm.js', which uses other things in the
#  build directory.
# Also, change the first or last attributes in index-build.html, so we'll
# see the change.

```

Now, use some server, such as xampp or the python http server,
 to run index-build.html in a browser.


## minification

I'll create a second version if index-build.html using some minified code.
```
cp -r build build1
cp index-build.html index-build1.html
```
The biggest piece of code in build directory is index-5dcbdb1a.js, 
at 32339 bytes.

I used one of the javascript minification services
 (https://javascript-minifier.com/) to minify index-5dcbdb1a.js
 In build1 directory, I'll replace index-5dcbdb1a.js with its minification.
 The minified size is 9072 bytes.

Modify index-build1.html
* change 'build' to 'build1'  (twice)
* change first name slightly so we'll see 'build1' in output of component.

Then run index-build1.html in browser.

## Conclusion
It is possible to get a completely compiled custom element with stencil.
When minified,  the amount of code for the element is quite small (only
about 9k of 'overhead').

This process could be automated.

It does NOT require uploading the component to npm.


