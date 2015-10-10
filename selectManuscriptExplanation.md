# Moving from "Add Item" to "Manifest List" to "Thumbnail"

In this write up, we describe what happens when, starting from the main screen (where you see a plus sign and "Add Item Link") you click the "Add Item Link", select a manifest, and are taken to the thumbnail screen

## Moving from the "Add Item" Screen to the select manifest list view

When you click on the "Add Item" link, the first thing that happens is that a click event is fired, for which the `$.Workspace` object is listening. When fired, the `$.Workspace` object triggers the event `addItem`. This functions identifies the "slot," in which the event was fired and looks for the parent object of the $.Workspace (i.e.$.Viewer) and requests that existing panels (i.e. workspace-container, workspacePanel, manifestPanel, bookmarkPanel) be toggled so that the div.workspace-container will be hidden and the `div.manifest-select-menu` will become visible.

At this point, a user will see the available manuscripts to select.

# Moving from the Manuscript Select View to the Thumbnail View

When the `manuscriptPanel` initialized, it appended a template  to the existing DOM tree that includes `div.select-results` containing an empty unordered list `ul.items-listing` where manuscriptListItems will be entered.

[unclear to me when ManifestListItems were created when manifests are not entered through the "add new object from Url box"]

Each <li> within this list is populated from an object called `$.ManifestListItem`.

On initialization of each item in the manifest list, a function called `fetchTplData` is called, which collects a bunch of information about the manifest and sets a `tplData` property on the `$.ManifestListItem` which is accessed in the template. This is used for labeling and describing the manifest in each list.

The initialization also identifies that manifestListElement (the empty <ul> element created in the manifestPanel) and prepend each manifestListItem into the DOM, at first hiding the created "<li>" and the fading it in for a nice effect.

When each manuscript loads a click is bound to each resulting <li> element. 

When you click on the the desired manifest an event bound the `$.Workspace` object called `.addWindow` is fired. This function takes as a parameter an object (referred to as `windowConfig`) with information needed to populate the thumbnail view, the most importance of which is the key `manifest` whose value `_this.manifest` points to the manifest to be loaded in the thumbnail view. 

The fired `.addWindow()` function does several things. 

First, it toggles the visibility of all panels besides the `$.Workspace` (represented in the DOM as `div.workspace-container`). It does this by changing the `overlayStates` of each to false (i.e. hidden) so that the `div.workspace-container` comes into view ([see lines 407-412](https://github.com/IIIF/mirador/blob/master/js/src/workspace.js#L407-L412)).

Next, it identifies the slot into which the manuscript should be loaded ([line 414-418](https://github.com/IIIF/mirador/blob/master/js/src/workspace.js#L414-L418)

Once identified, it creates a new object called $.Window, into which the same `windowConfig` object that was passed into the `.addWindow()` function triggered by the `$.manifestListItem` object. 

But how does the `$.Window` object actually get appended into the DOM tree?

The `$.Window` object gets appended into an already created $.Slot template. But when did this get created?

This actually already occurred when the `$.Workspace` object initialized. On initialization, it called a function named "calculateLayout" and this function created the number of slots specified in the relevant configuration information.

Now, on initialization, the `$.Window` object can be appended to a slot ([line 140](https://github.com/IIIF/mirador/blob/master/js/src/workspaces/window.js#L140)) specified back in the `$.Workspace.addWindow()` function where the `windowConfig.appendTo` was set to equal the `targetSlot.element`. 

The `$.Window` object gets most of the information it needs to populate its template and iniatated objects at [line 92](https://github.com/IIIF/mirador/blob/master/js/src/workspaces/window.js#L92), where it creates an `imageList` property by accessing the `$.Manifest` object and calling its `getCanvases()` function like so: `_this.manifest.getCanvases();`

Note that `$.Window` object template is responsible for building a number of `divs` that a user experiences when looking at particular manifest. This includes: the `div.manifest-info,` which is the top white bar above the images including the title of the manifest and various options to toggle to different overlays. Additionally, empty `divs` are created for `div.sidePanel` (where the table of contents are usually displayed) and the `div.view-container` (where the images actually show up). Within the view container, two other empty `divs` are created: a `div.overlay`, where the manifest info shows up when toggled, a `div.bottomPanel,` which (when not in thumbnail view) is where image thumbnails show up. Each of these empty divs are populated by further objects which were initiated through the `$.Window` object initiation sequence. 

Finally, to see how the thumbnail view gets populated, we have to go one step further and see where and how the `ThumbnailsView` was initiated.

When the `$.Window` object initialized, it checks to see what view should be initiated. 

A `$.Window` object has four possible overlay views, `ThumbnailsView`, `ImageView`, `ScrollView`, and `BookView`. By default, on initialization, the `$.Window` object is set (through the `currentFocus` property, which gets mapped to `focusState` variable within the initiation function) to focus on the the `ThumbnailsView`. So when a user originally selects a manifest from the `manifestList`, they are taken first to thumbnail view.

In [lines 147-163](https://github.com/IIIF/mirador/blob/master/js/src/workspaces/window.js#L147-L163), the $.Window object checks the `focusState` and fires the corresponding function. In this case, it fires the `toggleThumbnailsView()` function.

If the view is being created for the first time a `$.ThumbnailsView` object gets created while appending the thumbnail view to the empty `div.view-container' created by the `$.Window` object.

Within the `$.ThumbnailsView` initialization, a `loadContent()` function is called.

This function first creates an array of all the thumbnails ([line 43](https://github.com/IIIF/mirador/blob/master/js/src/widgets/thumbnailsView.js#L43)) called `tplData.thumbs`. Then it passes the `tplData` information into the template (containing the `div.thumbnail-view`) which is appended to the `div.view-container`.

At this point image thumbnails will have populated in the thumbnail view.






