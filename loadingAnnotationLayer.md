#What happens when you click the show annotation button?

When a user clicks on the "comments icon," they are clicking on a link found in the `$.Hud` template. When clicked, this fires an event bound to the class `.mirador-osd-annotations-layer`. 

This event in turn looks for the the `annoState` property and calls a function attached to the `annoState`.

So here we need to understand a bit of what the `annoState` property is and how it is created.

When the `$.Hud` object is created it first fires a function called `createStateMachine()` which creates a new `StateMachine` assigned to `this.annoState.` In the creation of this `StateMachine` a number of callback events are defined that will trigger depending on the current state of the viewer.

In the case of turning on annotations, it is the `displayOn` event that is triggered.

This event triggers a function called 
`ondisplayOn` which fires a jquery method called `publish()`, which triggers an event called `modeChange` for a specific window object and takes a second argument specifying the mode to change to.

We will see in moment that this "mode change" is the key to changing the UI experience. In actual fact, the sequence of events to load the annotations was initiated before the user ever clicked the show annotations button. 

So in order to see how these annotations got loaded, we have go back in time to when the single image view originally loaded.

When a single image view is chosen, for example from the `thumbnailView`, an `ImageView` object is initialized. 

A number of things happen here, but we are going to focus on how the `AnnotationLayer` gets created.

As part of the `imageView` creation a function called `createOpenSeadragonInstance()` gets created. 

This function does a lot of things to allow the image to load. 

But once this is completed another function, called `addAnnotationsLayer`, is fired.

This function initializes the `$.AnnotationsLayer` object ([widgets/annotationsLayer.js](https://github.com/IIIF/mirador/blob/master/js/src/widgets/annotationsLayer.js)). On initialize this function calls a function called `createRenderer()`; `createRenderer()` creates another object called the `$.OsdCanvasRenderer` ([annotations/osd-canvas-renderer.js](https://github.com/IIIF/mirador/blob/master/js/src/annotations/osd-canvas-renderer.js)). This object needs to be passed a number of pieces of information that can be traced back to the original `$.ImageView` call including the OSD in question, the id of the window to be populated, and most importantly the `annotationsList` to be used.

This `annotationsList` is acquired from the parent of the `$.ImageViewer` object which is the `$.Window` object.

When the `$.Window` object originally initiated, it executed a `getAnnotations()` function. The `getAnnotations` function looks for the Url of the `annotationsList` associated with the current canvas and requests the list using jquery's `get()` method. The returned list of annotations get add as an array of objects to the `$.Window.annotationsList` property.

This property is what then gets passed from the `$.ImageView` object to the `$AnnotationsLayer.object` and finally to the `$.OsdCanvasRenderer` object, now as just `list` and no longer as `annotationsList`.

It is finally here in the `$.OsdCanvasRenderer` that the blue boxes the user sees outlining an annotation area gets created. 

But it is not immediately obvious when this is done because `$.OsdCanvasRenderer` does not actually call any functions on initialization. However, it does have a render function. 

It the render function that takes the list array and, for each annotation within the list, creates a div and places that div on the designated regions of the open seadragon canvas. To do this, it uses the `parseRegion()` function to get the coordinates from the received Url and then uses the `getOsdFrame()` to create rectangle bounds on the OSD canvas.

[it's unclear to me from looking at `parseRegion` and `getOsdFrame` what the UI will do if the annotations is on the entire canvas and no specific coordinate region is mentioned.]

But the question remains, when did the `render()` function get fired? For this we have to return to the `$.AnnotationsLayer object.

At the end of the `createRenderer()` function which initiated the `$.OsdCanvasRenderer`, a final function named `modeSwitch()` is called. 

`modeSwitch()` is basically a switch to execute different actions depending on the `mode` of the `annotationsLayer`. 

Currently there are two possible modes, `displayAnnotations` and `editingAnnotations` and `default`. In the case of viewing annotations from a IIIF annotation list only rather than an annotation end point, the mode will be `displayAnnotations`. This mode will fire another function called `enterDisplayAnnotations()`. This function finally calls the `$.OsdCanvasRenderer` object just created as `this.renderer` and then calls the `$.OsdCanvasRenderer's` `render` method, like so, `this.renderer.render()` 

The the user click of the "show annotation button" occurs, 
the `publish()` method triggers an event called `modeChange` and when the mode changes, the bounding boxes are bound.


