# Ember-sourcemap-issue

This repo demonstrates https://github.com/ember-cli/ember-cli-uglify/issues/4

### Reproducting the Issue

To reproduce the issue, run:

```
ember build -e production
```

You should see this error:

```
C:\workspace\ember-sourcemap-issue>ember build -e production
version: 1.13.15
Build failed.
File: assets/vendor.js
Error: Invalid mapping: {"generated":{"line":85545,"column":-3029},"source":"bow
er_components/bootstrap/dist/js/bootstrap.js","original":{"line":1,"column":0},"
name":null}
    at SourceMapGenerator_validateMapping [as _validateMapping] (C:\workspace\em
ber-sourcemap-issue\node_modules\ember-cli-uglify\node_modules\broccoli-uglify-s
ourcemap\node_modules\uglify-js\node_modules\source-map\lib\source-map-generator
.js:271:15)
    at SourceMapGenerator_addMapping [as addMapping] (C:\workspace\ember-sourcem
ap-issue\node_modules\ember-cli-uglify\node_modules\broccoli-uglify-sourcemap\no
de_modules\uglify-js\node_modules\source-map\lib\source-map-generator.js:101:14)

    at C:\workspace\ember-sourcemap-issue\node_modules\ember-cli-uglify\node_mod
ules\broccoli-uglify-sourcemap\node_modules\uglify-js\node_modules\source-map\li
b\source-map-generator.js:72:19
    at Array.forEach (native)
    at SourceMapConsumer_eachMapping [as eachMapping] (C:\workspace\ember-source
map-issue\node_modules\ember-cli-uglify\node_modules\broccoli-uglify-sourcemap\n
ode_modules\uglify-js\node_modules\source-map\lib\source-map-consumer.js:155:16)

    at Function.SourceMapGenerator_fromSourceMap (C:\workspace\ember-sourcemap-i
ssue\node_modules\ember-cli-uglify\node_modules\broccoli-uglify-sourcemap\node_m
odules\uglify-js\node_modules\source-map\lib\source-map-generator.js:48:26)
    at Object.SourceMap (eval at <anonymous> (C:\workspace\ember-sourcemap-issue
\node_modules\ember-cli-uglify\node_modules\broccoli-uglify-sourcemap\node_modul
es\uglify-js\tools\node.js:24:4), <anonymous>:7575:52)
    at Object.exports.minify (C:\workspace\ember-sourcemap-issue\node_modules\em
ber-cli-uglify\node_modules\broccoli-uglify-sourcemap\node_modules\uglify-js\too
ls\node.js:102:38)
    at UglifyWriter.processFile (C:\workspace\ember-sourcemap-issue\node_modules
\ember-cli-uglify\node_modules\broccoli-uglify-sourcemap\index.js:121:27)
    at C:\workspace\ember-sourcemap-issue\node_modules\ember-cli-uglify\node_mod
ules\broccoli-uglify-sourcemap\index.js:62:16
```

Note that removing the following line from `ember-cli-build.js` fixes the issue:

```
  app.import('bower_components/sprintf/dist/sprintf.min.js');
```

(The error message is cryptic because sprintf, NOT bootstrap is the problem)


### My Theory

ember-cli-uglify requires 3rd-party bower libs to have their magic sourcemap at the BEGINNING of their source, so that Ember CLI can concatenate them like this:

```
//# sourceMappingURL=lib1.map
lib1.js code

lib2.js code   // this lib doesn't have a sourcemap

//# sourceMappingURL=lib3.map
lib3.js code
```

If a lib declares the magic comment at the END of its file, the following happens:

```
lib1.js code

//# sourceMappingURL=lib1.map      ember gets con
lib2.js code
```

Because Ember CLI evaluates the maps on the CONCATENATED file, here it apparently gets confused and thinks the source map is for lib2 when it's actually for lib1.

This explains why the error says `bootstrap.js` as the offending lib, when infact its `sprintf`.

My sample repo uses `sprintf.js` as an offender which has magic comment at the end. If you manually edit the `sprintf.js` to have the magic comment at the beginning, everything works.
