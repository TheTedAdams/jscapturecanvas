# Jscapturecanvas aka jsCanvas #

[Canvg](http://code.google.com/p/canvg/) is a SVG->Canvas translator which takes an SVG input and directly writes into a html5 canvas. Jscapturecanvas works with canvg to "compile" the SVG input into a javascript object that can be loaded at a later time. This obviates the need for loading canvg and the SVG files at deployment time, as well as producing the opportunity to change the runtime environment when the object is instantiated, like substituting colors for patterns, scaling, etc.

## Using jsCanvas ##

### jsCanvas Constructor ###

Create a new jsCanvas which can be used to capture the javascript needed redraw
the SVG file at a later time without the need of either the SVG, or the rendering
javascript (canvg).
```
new jsCanvas (objectname, [width, height]);

objectname: the name of the object that will be created when the javascript is loaded/evaled 
[width, height]: optional values to scale the object to fixed values at compile time; 
the default is to let canvg figure out the SVG height and width.
```

Normally, you will then render() to compile the SVG, and toString () to get the resulting
javascript object. This object can be stored and loaded at a later time in your production code (typically).

### jsCanvas.compile ###

Compile the svg file into a javascript object for later use.

```
compile (svg [[, oncomplete], canvgopts]);

svg: a string containing SVG to be compiled
oncomplete: function to call when done. note that given images, compile complete asynchronously
canvgopts: alternate set of options to feed to canvg... probably don't want to change the default.
```
### jsCanvas.toString ###
```
toString ()
```

Return the string representation of the javascript object created above. Normally you'll probably want to capture the output somehow and store it on your webserver. For example, you might consider doing an AJAX POST to your server of the compiled javascript object.

See the example code [here](http://mtcc.com/site/jscanvastest.html) for how this all works.

## Using Compiled Objects at Runtime ##

Once the SVG is compiled into javascript, stored on your web server and loaded onto
your webpage, you instantiate the object with:

```
var myobj = new myObject (ctx2d);
```

assuming the "objectname" above was set to 'myObject'; ctx2d is the 2d context of
a Canvas tag (eg getContext('2d')) that you want to draw into.

An optional feature is that you can capture all of the sets to ctx2d's fields by overriding the setter method of the object. eg:

```
myobj.setter = function (ctx, key, value) { ctx [key] = value; } 
```
Obviously the setter function can be more elaborate and transform fill colors, etc.
Another niceism is that the object contains the bounding box of the SVG file in
myobj.inwidth and myobj.inheight. You can use them to scale your object at runtime
to fit your canvas if you like, for example:

```
ctx2d.setTransform(1, 0, 0, 1, 0, 0);			// reset transforms to base
ctx2d.scale (200/myobj.inwidth, 400/myobj.inheight);	// squeeze this into 200x400
```

once you've set everything up, you then just render the object and you're done.

```
myobj.render ();
```

If you have images in the canvas (including patterns), they need to be completely loaded lest pattern fills, etc will not work properly. You can check to see if the images are loaded by calling the static method:

```
if (! myObject.imagesLoaded ()) {
    // do something to try again later when they are loaded, such as setTimeout.
}
```

Since the images are inlined as data: url's, they should be pretty quick to load, but it's not guaranteed.