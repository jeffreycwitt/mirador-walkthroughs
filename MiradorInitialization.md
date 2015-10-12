#Mirador Initialize Walk Through

## The main-menu-bar and mirador-viewer wrapper

When a `$.Mirador` object is initiated from the `index.html` file, this initializes the `$.Viewer` ([viewer.js](https://github.com/IIIF/mirador/blob/master/js/src/viewer.js)) using any config information specified in the `$.Mirador` object.

The initialization of the `$.Viewer` object then does a lot of work.

First, it builds the `$.mainMenu` object (that is, if `showMainMenu` is set to true) ([line 61](https://github.com/IIIF/mirador/blob/master/js/src/viewer.js#L61)) and adds this as a property to the $.Viewer object.

The `$.mainMenu` object is defined in [/viewer/mainMenu.js](https://github.com/IIIF/mirador/blob/master/js/src/viewer/mainMenu.js)). On [line 37](https://github.com/IIIF/mirador/blob/master/js/src/viewer/mainMenu.js#L37), the object initialization function uses this jquery `appendTo()` method to append the main menu into the DOM tree as `div.mirador-main-menu-bar` positioned as a direct child of `div.viewer` or whatever `div` id was specified in the original `$.Mirador` object setup. 

As we will see, this process of appending `div` elements (which correspond to javascript objects) to existing `div` elements is the usual method in which Mirador constructs the html page that the user sees. However, the next step is an exception.

After adding the `$.mainMenu`, the viewer initialization loads the viewing area, `div.mirador-viewer` as a following-sibling of `div.main-menu-bar`.  It does this by creating a jquery object as property of `$.Viewer` called canvas (`this.canvas`) giving it the class name `mirador-viewer` and appending (again using the jquery method `.appendTo()` to the the `$.Viewer` object element. This `div` is used as a container to separate everything that follows from the `div.main-menu-bar`.

The canvas position is then slightly adjusted if the original Mirador object set mainMenu to false.

##Workspace configuration

The next step is to create a workspace within the mirador viewer-wrapper that was just created. The same approach is used as before. 

A `$.Workspace` object ([workspace.js](https://github.com/IIIF/mirador/blob/master/js/src/workspace.js)) is added as a property to the `$.Viewer` object (as `this.workspace`) and a number of parameters are passed to the initialization function including a target element for the jquery `appendTo()` function. In this case `this.element.find('.mirador-viewer')`.

Here one should not confuse the parent object of the `$.Workspace` object (the `$.Viewer` object) with the parent element `div.mirador-viewer` in the DOM.

After this, three more "panels" get built that, in the DOM tree, are siblings of the `$.Workspace` object element.

These are the `workspacePanel`, which in the DOM tree is appended as `div.workspace-select-menu.` This is the view you see when you are creating a workspace, i.e. selecting how many windows (or `.layout-slot`s) you want in your workspace (or `div.workspace-container`)

Next is the `manifestPanel`. This is the view seen when selecting from multiple manifests, which is `div.manifest-select-menu` in the DOM tree.

Finally the `bookmarkPanel` object is created (`this.bookmarkPanel`) which is appended as following-sibling to the previous two panels and workspace-container in the DOM tree.

## Final Actions

At this point the complete DOM tree at initialization has been constructed. 

Three final things happen upon initialization. 

First, since there is now `div.workspace-container` along with three different panels created within the `div.mirador-viewer,` Mirador needs to decide which panel to display as the default.

The default is set to the workspace (or `div.workspace-container`) while the `workspacePanel`, `manifestPanel`, and `bookmarkPanel` remain out of view.

Second, it binds events to several different functions included in the `viewer.js` file.

Third, it begins retrieving `manifestData` from the manifest urls supplied in the original `$.Mirador` object. 

For each supplied manifest a `$.Manifest` object is created which upon initialization requests the `.json` data using jquery's ajax method. The loaded manifest objects eventually get placed in array which is attached to the `$.Viewer` object with the property `.manifests` which becomes available for later use.