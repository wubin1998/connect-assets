# connect-assets

[![Build Status](https://travis-ci.org/adunkman/connect-assets.png?branch=master)](https://travis-ci.org/adunkman/connect-assets)

Transparent file compilation and dependency management for Node's [connect](https://github.com/senchalabs/connect) framework in the spirit of the Rails 3.1 asset pipeline.

## The state of the package

As of February 21, 2013, @adunkman became the maintainer of this package. The game plan is as follows:
- Address critical issues with version 2.x (master branch) to take care of the open pull requests/issues.
- Begin a version 3.x (v3 branch) that introduces stronger tests and code structure to make contributing easier to manage, reducing dependencies as possible.

## Plans for version 3.0

* Rewrite in JS
* Cleaner code
    * Shorter files
    * Better defined responsibilities
* No singleton
* Replace connect-file-cache
* Look for replacment for Snockets
* Remove dependence on underscore
* Remove dependence on mime
* Write tests in mocha
* Remove Cakefile

## What?

connect-assets can:

1. Serve `.coffee` ([CoffeeScript](http://coffeescript.org)) files as compiled `.js`
1. Concatenate `.coffee` and `.js` together using [Snockets](https://github.com/TrevorBurnham/snockets)
1. Serve `.styl` ([Stylus](http://learnboost.github.com/stylus/)) as compiled `.css` with
    -  [nib](https://github.com/visionmedia/nib)
    -  [Twitter Bootstrap](https://github.com/shomeya/bootstrap-stylus)
1. Serve `.less` ([Less](http://lesscss.org/)) as compiled `.css` with
    - [Twitter Bootstrap](https://github.com/twitter/bootstrap)
1. Serve files with an MD5 hash suffix and use a far-future expires header for maximum efficiency
1. Avoid redundant git diffs by storing compiled `.js` and `.css` files in memory rather than writing them to the disk—except when you want them (e.g. for deployment to a CDN).

## How?

First, install it in your project's directory:

    npm install connect-assets

Also install any specific compilers you'll need, e.g.:

    npm install coffee-script
    npm install stylus
    npm install nib
    npm install bootstrap-stylus
    npm install less

Then add this line to your app's configuration:

    app.use require('connect-assets')()

Finally, create an `assets` directory in your project and throw all your `.coffee` files in /assets/js and `.styl`, `.less` files in /assets/css.

### Markup functions

connect-assets provides two global functions named `js` and `css`. Use them in your views. They tell connect-assets to do any necessary compilation, then return the markup you need. For instance, in a [Jade template](http://jade-lang.com/), the code

    != css('normalize')
    != js('jquery')

(where `!= is Jade's syntax for running JS and displaying its output) results in the markup

    <link rel="stylesheet" href="/css/normalize.css" />
    <script src="/js/jquery.js"></script>

For your purposes you can pass additional attributes to helpers `js` and `css`.

    != css('normalize', { media: 'print', 'data-confirm': 'Are you sure to do this?', 'data-delete': true })

Result:
    
    <link rel="stylesheet" href="/css/normalize.css" media="print" data-confirm="Are you sure to do this?" data-delete />


### Sprockets-style concatenation

You can indicate dependencies in your CoffeeScript files using the Sprockets-style syntax

    #= require dependency

(or `//= require dependency` in JavaScript). When you do so, and point the `js` function at that file, two things can happen:

1. By default, you'll get multiple `<script>` tags out, in an order that gives you all of your dependencies.
2. If you passed the `build: true` option to connect-assets (enabled by default when `process.env.NODE_ENV == 'production'`), you'll just get a single tag, wich will point to a JavaScript file that encompasses the target's entire dependency graph—compiled, concatenated, and minified (with [UglifyJS](https://github.com/mishoo/UglifyJS)).

If you want to bring in a whole folder of scripts, use

    #= require_tree dir

See [Snockets](http://github.com/TrevorBurnham/snockets) for more information.

**Note:** CSS concatenation is not supported by connect-assets directly, because Stylus and Less already do a fine job of this. Stylus and Less are basically supersets of CSS, so just rename your `.css` files to `.styl` or `.less` and learn about the @import ([Stylus](http://learnboost.github.com/stylus/docs/import.html), [Less](http://lesscss.org/#-importing)) syntax.

## Options

If you like, you can pass any of these options to the function returned by `require('connect-assets')`:

* `src` (defaults to `'assets'`): The directory assets will be read from
* `helperContext` (defaults to `global`): The object the `css` and `js` helper functions will attach to
* `buildDir` (defaults to `builtAssets`): Writes built asset files to disk using this directory in `production` environment, set to `false` to disable
* `servePath` (defaults to ""): Paths in generated tags will be prefixed by `servePath`.  Useful when working with reverse proxies.
* ... see the source (`src/assets.coffee`) for more.

### Using `helperContext` 

You can also set the "root path" on the `css` and `js` helper functions (by default, `/css` and `/js`), e.g.

    css.root = '/stylesheets'
    js.root  = '/javascripts'

To override these roots, start a path with `'/'`. So, for instance,

    css('style.css')

generates

    <link rel='stylesheet' href='/css/style.css'>

while

    css('/style.css')

gives you

    <link rel='stylesheet' href='/style.css'>
    
### Using `servePath`
    
In addition to setting the "root path" where assets can be found by the server, you can also "mount" your assets behind a base location path by setting `servePath`, so that path references are generated correctly for tags.  This is especially useful when working with reverse proxies, where the server may be receiving external requests for assets at a different path, e.g.

    require('connect-assets')(
      servePath: '/foo'
    )
   
then,

    css('/style.css')
   
gives you

    <link rel='stylesheet' href='/foo/css/style.css'>

While the server will still expect that external requests for `'/foo/css/style.css'` will be rewritten for `'/css/style.css'`  This only applies to absolute paths.

## Generated documentation 

There is generated documentation (created with [docco](http://jashkenas.github.com/docco/)) available [here](http://adunkman.github.com/connect-assets/).

## Credits

Borrows heavily from Connect's [compiler](https://github.com/senchalabs/connect/blob/1.6.4/lib/middleware/compiler.js) and [static](https://github.com/senchalabs/connect/blob/1.6.4/lib/middleware/static.js) middlewares, and of course sstephenson's [Sprockets](https://github.com/sstephenson/sprockets).

Look at these [awesome people](https://github.com/adunkman/connect-assets/contributors) who make this project possible.
