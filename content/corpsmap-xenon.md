## CorpsMap-Xenon

CorpsMap-Xenon is a flexible mapping application built on React.js, OpenLayers3 and Cesium.js.  The tool provides 2d and 3d viewing of data and a number of basic plugins for interacting with data and services.

Library specific documentation for help with the major components used in CorpsMap-Xenon are available here:

* [React.js](https://facebook.github.io/react/)
* [OpenLayers3](http://openlayers.org/)
* [Cesium.js](https://cesiumjs.org/)
* [Ol3-Cesium Integration Propject](http://openlayers.org/ol3-cesium/)

As programs and developers adopt CorpsMap-Xenon for their own use, more plugins will be needed and those should follow the architecture laid out below.

There are two ways to develop plugins for CorpsMap-Xenon.

1. As an integrated plugin that is compiled into the base corpsmap-xenon.js library.
2. As a stand-alone .js file that can be included in any html document after the corpsmap-xenon.js library.

Option 1 will be used for functionality that should be rolled into the primary project to make the plugin available to everyone using the library.  Anyone can pull the project down and develop plugins this way, pushing it back to the main project using a pull request a the project repository in DI2E.  This process is documented [here]() (coming soon), please review the contribution docs carefully before submitting a pull request.

Option 2 makes it easy for anyone to put together a plugin and include it in a single deployment of corpsmap-xenon.js.  This way the developer doesn't have to get into the nitty-gritty of the whole development setup.

We'll go through both processes, but the main difference is that plugins developed using Option 1 will be CommonJS modules that are included in the build process and should `require()` all dependencies.  Plugins using Option 2 will have the main dependencies that they will need available in the global name space, exposed by the corpsmap-xenon.js library.

### Global Variables

To enable Option 2 style plugins, we expose a number of global variables that allow plugins to hook into the application.  Option 1 style plugins should use the CommonJS `require()` syntax to access libraries rather than using the global variables.

#### Global Variables

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
window.CorpsMap
window.React
window.ReactRouter
  // includes
  window.ReactRouter.Router
  window.ReactRouter.Route
  window.ReactRouter.Link
```

```python
# Client API not available in Python
```

`window.CorpsMap` is the primary application API and exposes the same object that [`require('ampersand-app')`](#ampersand-app) does when creating Option 1 plugins.

`window.React` exposes the React.js API to allow creation of new components.

`window.Router` and it's children expose the [react-router](https://github.com/reactjs/react-router) API, react-router is used to manage the plugins that expose UI components using the left sidebars.  Most plugins will only need to use the `window.ReactRouter.Link` component.


### Application API

The application API is exposed when using the `window.CorpsMap` global variable or when using `var app = require('ampersand-app')` in CommonJS modules inside the project.

#### Examples

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
// coming soon
```

```python
# Client API not available in Python
```


Key | Description
--- | ---
[`CorpsMap.actions`](#actions) | Application level [Reflux](https://github.com/reflux/refluxjs) actions, more information available [below](#actions).
`CorpsMap.history` | Stores the navigation history of the `React.Router` that drives the left-side panel system.
`CorpsMap.mapProjection` | Utility reference to the `ol.proj.Projection` with the EPSG code of 3857, or web mercator, used by the map frame.
`CorpsMap.geoProjection` | Utility reference to the `ol.proj.Projection` with the EPSG code of 4626, or WGS84, the most common Lat-Lon projection that data will come in.
[`CorpsMap.optionsStore`](#options) | [Reflux](https://github.com/reflux/refluxjs) store that maintains user options exposed by plugins, more information about options available [below](#options).
`CorpsMap.componentStore` | [Reflux](https://github.com/reflux/refluxjs) store that contains references to each of the React components exposed by plugins.  The component store is queried by each of the plugin regions to get the components that need to be placed in each of the regions.
[`CorpsMap.layerStore`](#map-layers) | [Reflux](https://github.com/reflux/refluxjs) store used to manage the map layers that are added to the map and exposed through the layer tree component.  See the section below about [map layers](#map-layers) for more information.
`CorpsMap.globalStateStore` | [Reflux](https://github.com/reflux/refluxjs) store used to manage generic state for the application.  More infomation can be found in the [global state management](#global-state-management) section.
`CorpsMap.ol2d` | Reference to the [OpenLayers3](#http://openlayers.org/) map object that is exposed by calling `new ol.Map()`.  This is the primary 2d canvas.
`CorpsMap.ol3d` | Reference to the OpenLayers3 wrapper on top of [CesiumJS](https://cesiumjs.org/) used for 3d rendering.  Exposed by calling `new olcs.OLCesium()`.
`CorpsMap.init()` | Function used to initalize the app and render the main component to the DOM in the `<div id="root"></div>` node.  Used internally, should not be called more than once.

### Actions

Actions are how state is changed by components at lower levels of the application.  If we think of the application as a tree of components, the state is managed at the top of the tree and changes cascade down through the different components.  To make a change at the top, lower level components have to initiate actions, which are basic function calls that the state stores listen to and act upon.

The available actions are:

Action | Description
--- | ---
`register` | Used to register components as discussed in the [Plugin Registration](#plugin-registration) section.
`unregister` | Used to unregister a component and remove it from the plugin regions.
`addUpdateOption` | Used to add an option to the options list, usually done during plugin registration. See example at right.  More information about options available [below](#options).
`addlayer` | Used to add a layer to the layer store and to the layer tree view.  See the section below about [map layers](#map-layers) for more information.  Function takes a layer properties object as in the example to the right.  Note that you still have to add the layer to the map separately.
`removelayer` | Used to remove a layer from the layer store and from the layer tree view, as well as from the map. Function takes a layer id.
`setvisible` | Used to set visibility on a layer in the map. Function takes a layer id.
`zoomto` | Used to zoom the map to the extents of a layer if the extents are available, usually requires vector data to work.
`statechange` | Used to set a key:value on the generic state object, can be used by plugins to manage and listen for state such as changing between 2d and 3d.

#### Example Actions

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
// app = window.CorpsMap

// Adding a user option
var newOption = {
  _id: pluginKey,
  value: 'enable',
  defaultValue: 'enable',
  ui: true,
  group: 'Enable/Disable Components',
  label: plugin.pluginName,
  type:'toggle',
  values: ['enable', 'disable']
}
app.actions.addUpdateOption(newOption)

// Adding a layer to the app, and the map canvas.
app.actions.addlayer({
  id: 'some-layer:id',
  type: 'point',
  path: 'test',
  label: 'TEST layer',
  visible: false,
  layer: layer
});
app.ol2d.addLayer(layer)
```

```python
# Client API not available in Python
```

### Options

### Map Layers

### How the Plugins Work

CorpsMap-Xenon plugins are made up of one or more React.js components and follow the internal [React.js lifecycle](https://facebook.github.io/react/docs/component-specs.html) just like any other React component.

What makes these different is that they must register with the application and tell it where they would like to show up in the user interface.

Each plugin must have a place where it calls `CorpsMap.actions.register()` to register its component(s) like in the example to the right.  The register function takes an object that provides the name of the plugin and an array of components associated with the plugin.  Each component object requires a reference to the React component and a role that describes which plugin region that component should be placed into.  Other options can also be passed in the component object to be stored and recalled later.  There are more comprehensive examples below.

Plugins can register components into a number of plugin regions in the UI (plugins can also expose their own plugin regions, more on that later...)

#### Plugin Registration

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
window.CorpsMap.actions.register({
  pluginName: 'Plugin Name',
  components:[{
    component: PluginToolIcon,
    role:'layer-details-toolbar'
  }]
});
```

```python
# Client API not available in Python
```

### Plugin Regions

There are a number of regions in the UI where plugins can attach components, and some plugins may attach more than one component for example if there is a sidebar button that opens a panel in the primary panel region you would register both components as in the example to the right.

Note that in the example the path option tells the router when to open the right component in the primary panel and must match the path provided in the link on the icon component.

#### Multi-Component Plugin Registration

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
var plugin = {
  register: function(){
    CorpsMap.actions.register(
      this.iconComponent,
      {role: 'primary-sidebar', path:'/'}
    );
    CorpsMap.actions.register(
      this.primaryComponent,
      {role: 'primary-plugin', path:'/primarycomponent'}
    );
  },
  iconComponent: React.createClass({
    // create your sidebar button here
  }),
  primaryComponent: React.createClass({
    // create your primary panel content here
  })
}

plugin.register();
```

```python
# Client API not available in Python
```

Components can be registered with the following roles that will drive where they are displayed in the UI:

* `'primary-sidebar'` displays an icon component in the upper-left of the primary sidebar at the left side of the screen.  This should be used for tools that need a panel to expand to be functional.

![primary-sidebar screenshot](./img/primary-sidebar.png)

* `'primary-sidebar-bottom'` displays an icon in the left sidebar but at the bottom of the region, like the options and settings tool.

![primary-sidebar-bottom screenshot](./img/primary-sidebar-bottom.png)

* `'primary-plugin'` displays a panel component inside the primary panel that expands when a primary icon is clicked. For example the Map Layers panel shown below:

![primary-plugin screenshot](./img/primary-plugin.png)

* `'upper-right-toolbar'` displays icons for tools that interact with the map on a single click.  These tools do not natively have access to any panels or other UI components.

![upper-right-toolbar screenshot](./img/upper-right-toolbar.png)

* `'lower-right-toolbar'` displays components that need a little horizontal UI, but do not have access to more complicated panel UI components.  Icons can be used in addition to more complicated tools such as the geocoder.

![lower-right-toolbar screenshot](./img/lower-right-toolbar.png)

* `'basic-toolbar'`  displays icons for tools that have a toggled state, they are either on or off and interact with the map when turned on.

![basic-toolbar screenshot](./img/basic-toolbar.png)

* `'layer-details-toolbar'` displays icon based tools that should act on an individual layer inside that layer's details panel.

![layer-details-toolbar screenshot](./img/layer-details-toolbar.png)

* `'layer-details-config'` displays more interactive tools that should act on an individual layer inside the panel of that layer's detail panel.

![layer-details-config screenshot](./img/layer-details-config.png)

### Option 1 Plugin Architecture

Option 1 plugins are included in the core project and compiled into corpsmap-xenon.js.  When writing these plugins we can leverage the build system and write CommonJS modules.  The basic requirements for a plugin are the same in either option, you just orgainize your code a little differently.

A plugin can have one or more React components that will be placed in the UI as well as any number of internal functions that can be leveraged by the plugin.

The plugin module must export a function called `.register()` that will register any components with their proper role and path options as applicable.

Components can be created as part of the main object or externally and only referenced in the register function.

To add the plugin to the application, you add the require statement to the main app module at `src/corpsmap.js`. (this is subject to change)

#### Plugin with Components included in the module

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
var React = require('react');

module.exports = {
  title:'switch-dimensions',

  register: function () {
    CorpsMap.actions.register(this.iconComponent, {
      role: 'upper-right-toolbar'
    });
  },

  iconComponent: React.createClass({
    displayName: 'switch-dimensions',

    getInitialState: function(){
      return {
        threeD:false
      }
    },

    onClick: function(){
      this.setState({ threeD:!this.state.threeD });
    },

    componentDidUpdate: function(){
      CorpsMap.ol3d.setEnabled(this.state.threeD);
      CorpsMap.actions.statechange(this.state);
    },

    render: function(){
      var selectedClass = this.state.threeD ? 'selected' : '';
      return (
        <div id='switch-dimensions' className={'sb-primary-icon ' + selectedClass} title="Switch Dimensions" onClick={this.onClick} >
          <i className="mdi mdi-google-earth"></i>
        </div>
      );
    }
  })
};
```

```python
# Client API not available in Python
```

#### Plugin with Components outside of the module

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
var React = require('react');
var app = require('ampersand-app');

var TrashTool = React.createClass({
  onClick: function(){
    app.history.push('/layertree')
    app.actions.removelayer(this.props.id)
  },
  render: function(){
    return (
      <span onClick={this.onClick} title="Delete Layer" className="layer-details-tool">
        <i className="mdi mdi-delete"></i>
      </span>
    )
  }
})

module.exports = {
  register: function(){
    app.actions.register(TrashTool, {role: 'layer-details-toolbar'});
  }
}
```

```python
# Client API not available in Python
```

### Option 2 Plugin Architecture

Option 2 plugins can be placed in a script tag after `corpsmap-xenon.js` and need to call register inside their own script, that will add them to the component store and the UI.

The example to the right show what the second Option 1 example would look like if refactored to be an Option 2 style plugin.

React is available at `window.React` and the app global is available as `window.CorpsMap`.

#### Option 2 style Plugin

```curl
# Client API not available in cURL
```

```bash
# Client API not available in bash
```

```javascript
var TrashTool = React.createClass({
  onClick: function(){
    CorpsMap.history.push('/layertree')
    CorpsMap.actions.removelayer(this.props.id)
  },
  render: function(){
    return (
      <span onClick={this.onClick} title="Delete Layer" className="layer-details-tool">
        <i className="mdi mdi-delete"></i>
      </span>
    )
  }
})

var removeLayerPlugin = {
  register: function(){
    CorpsMap.actions.register(TrashTool, {role: 'layer-details-toolbar'});
  }
}

removeLayerPlugin.register();
```

```python
# Client API not available in Python
```
