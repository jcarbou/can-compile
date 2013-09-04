# can-compile

NodeJS module that compiles [CanJS](http://canjs.us/) EJS and Mustache views into a single JavaScript file for lightning fast
production apps.

With NodeJS installed, just run NPM:

> npm install can-compile -g

## Command line

The `can-compile` command line tool takes a list of files (by default all `*.ejs` and `*.mustache` files in the current folder)
or a list of [filename patterns](https://github.com/isaacs/minimatch) and writes the compiled views into an `out` file
(default: `views.production.js`).

__Examples:__

Compile all EJS and Mustache files in the current folder and write them to `views.combined.js`:

> can-compile --out views.combined.js

Compile `todo.ejs` using CanJS version 1.1.2, write it to `views.production.js`:

> can-compile todo.ejs --can 1.1.2

Compile all EJS files in the current directory and all subdirectories and `mustache/test.mustache`.
Write the result to `views.combined.js`:

> can-compile **/*.ejs mustache/test.mustache --out views.combined.js

## Grunt task

can-compile also comes with a [Grunt](http://gruntjs.com) task so you can easily make it part of your production build.
Just `npm install can-compile` in you project folder (or add it as a development dependency).
The following example shows a Gruntfile that compiles all Mustache views and then builds a concatenated and minified `production.js`
of a CanJS application:

```javascript
module.exports = function (grunt) {

  // Project configuration.
  grunt.initConfig({
    cancompile: {
      dist: {
        src: ['**/*.ejs', '**/*.mustache'],
        out: 'production/views.production.js'
      },
      legacy: {
        src: ['**/*.ejs', '**/*.mustache'],
        out: 'production/views.production.js',
        options: {
          version: '1.1.2'
        }
      }
    },
    concat: {
      dist: {
        src: [
          '../resources/js/can.jquery.js',
          '../resources/js/can.view.mustache.js',
          'js/app.js', // You app
          '<%= cancompile.dist.out %>' // The compiled views
        ],
        dest: 'production/production.js'
      }
    },
    uglify: {
      dist: {
        files: {
          'production/production.min.js': ['<%= concat.dist.dest %>']
        }
      }
    }
  });

  // Default task.
  grunt.registerTask('default', ['cancompile', 'concat', 'uglify']);

  grunt.loadNpmTasks('can-compile');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-concat');
};
```

## Programmatically

You can compie files directly like this:

```javascript
var compiler = require('can-compile');

compiler.compile('file.ejs', function(error, output) {
  output // -> compiled `file.ejs`
});
```

Passing an object as the first parameter allows you the following configuration options:

- `filename` {String}: The name of the file to be compiled
- `version` {String} (default: `latest`): The CanJS version to be used
- `log` {Function}: A logger function (e..g `console.log.bind(console)`)
- `normalizer` {Function}: A Function that returns the normalized path name

```javascript
compiler.compile({
  filename: 'file.ejs',
  log: console.log.bind(console),
  normalizer: function(filename) {
    return path.relative(__dirname, filename);
  },
  version: '1.1.6'
}, function(error, output) {
  output // -> compiled `file.ejs`
});
```

## Loading with RequireJS

To use your pre-compile views with [RequireJS](http://requirejs.org/) just shim the production file with
`can/view/mustache` and/or `can/view/ejs` (depending on what you are using) as the depedencies and
then load it before your main application file:


```js
require.config({
  shim: {
    'views.production': {
      deps: ['can/view/ejs', 'can/view/mustache']
    }
  }
});

require(['views.production', 'app/index']);
```

## Note

Always make sure that the output file is in the same folder as the root level for the views that are being loaded.
So if your CanJS applications HTML file is in the `app` folder within the current directory use a filename within
that folder as the output file:

> can-compile --out app/views.production.js

## Changelog

__0.3.0:__

- Allows compilation for different CanJS versions

__0.2.1:__

- Switched to plain JSDom
- Update to CanJS 1.1.5
- Verified Node 0.10 compatibility

__0.2.0:__

- Grunt 0.4.0 compatibility
- Added Travis CI

__0.1.0:__

- Initial release

[![Build Status](https://travis-ci.org/daffl/can-compile.png?branch=master)](https://travis-ci.org/daffl/can-compile)
