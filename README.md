# Esri Developer Summit 2016 - Ember demo using Esri javascript application

This project comes with minimal settings.
It serves as the base for the demo below.

## Prerequisite
Your machine must have installed:
* node 0.12.7 or higher
* bower (latest): `npm install -g bower`
* [ember-cli](http://ember-cli.com/): `npm install -g ember-cli`

All the commands described below have to be executed from your terminal window from inside the project folder.

Note that, as today (03/09/16), we are using the beta esri js api 4.0.
You may want to update this reference in the future.

## Setting up the project for using Esri js api

### Step 1

`ember install ember-cli-amd`

Ember-cli comes with its own loader that is incompatible with AMD used Esri js api.
We created an ember-cli addon that solves this issue [ember-cli-amd](https://github.com/Esri/ember-cli-amd)

### Step 2

Update the ember-cli-build.js:

```
/*jshint node:true*/
/* global require, module */
var EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function (defaults) {
  var app = new EmberApp(defaults, {
    amd: {
      loader: 'http://jsdev.arcgis.com/4.0/',
      packages: [
        'esri', 'dojo', 'dojox', 'dijit',
        'put-selector', 'xstyle', 'dbind', 'dgrid'
      ]
    }
  });

  return app.toTree();
};
```

### Step 3

Update the app/index.html by adding the cdn based style after `{{content-for "head"}}`
```
{{content-for "head"}}
<link rel="stylesheet" href="//jsdev.arcgis.com/4.0/esri/css/main.css">
```

### Serve your app and test

Open a new terminal window in your project folder and type `ember serve`. This will build the app and serve it under `http://localhost:4200`. It will keep building the app and refresh the browser tab as you update your code.

Open a browser tab on: `http://localhost:4200`. If everything is right, you should see the following text on you tab: Hello Dev Summit geeks!

## Part 1: Creating a map component
We want more than just a nice greeting test on the screen.
The goal is to get an esri scene view to display.

### Create the component

`ember g component esri-map`

This will create a new file under app/components/esri-map.js

Update this file as following:
```
import Ember from 'ember';
import Map from 'esri/Map';
import SceneView from 'esri/views/SceneView';

export default Ember.Component.extend({

  didInsertElement() {
    var map = new Map({
      basemap: "streets"
    });
    this.view = new SceneView({
      container: this.element,
      map: map
    });
  }
});
```

### Update the app/templates/application.hbs

Replace all the content of the template with:

```
{{esri-map}}
```

### Test
if your `ember serve` is still running and your browser tab is still open on `http:\\localhost:4200`, you should see a scene view (globe).


## Part 2: Improve the map component
The goal is to make our component more controllable...
One of the power of ember is to set us up on the road of web components.


### Add a 2d/3d component property
Goal: Control the type of map (2d, 3d)

Update the app/components/esri-map as follow:

```
import Ember from 'ember';
import Map from 'esri/Map';
import SceneView from 'esri/views/SceneView';
import MapView from 'esri/views/MapView';

export default Ember.Component.extend({

  type: '3d', // default value

  didInsertElement() {
    var map = new Map({
      basemap: "streets"
    });

    let type = this.get('type');
    switch (type) {
    case '3d':
      this.view = new SceneView({
        container: this.element,
        map: map,
        scale: 50000000,
        center: [-101.17, 21.78]
      });
      break;

    case '2d':
      this.view = new MapView({
        container: this.element,
        map: map,
        zoom: 4,
        center: [15, 65]
      });
      break;
    }
  }
});
```

Update the app/templates/application.hbs as:
```
{{esri-map type='2d'}}
```

### Test
As you change the component property `type` to 2d or 3d, you should see your app changing.


### Add config support
The previous example illustrate how ember component can be controlled by properties.
However, this example is kind of limited to only change form 2d to 3d but the Scene or Map views cannot really be configured.

An ember app is often made of a collection of 'routes'. Routes are the entry points of your application.
In our scenario, we will, have only one route, the application route. More sophisticated applications can have many routes and sub-routes.

Create new application route `ember g route application`
Note this command will ask you if the app/templates/applications.hbs should be overwritten. You can answer 'yes' or 'no', it doesn't matter since we will update it.

The command will create a new file app/routes/applications.js.

A route in ember is responsible to define the model used by the route controller and components (data for the route).

Update the app/routes/application.js:
```
import Ember from 'ember';
import Map from 'esri/Map';

export default Ember.Route.extend({

  model() {
    var map = new Map({
      basemap: "streets"
    });

    return {
      type: '3d',
      map: map,
      camera: {
        position: [10, 53.52, 2820],
        tilt: 50
      }
    };
  }
});
```

We are creating a new Map in the route and we are returning an object (our model) with different proeprties.

Update app/components/esri-map.js:
```
import Ember from 'ember';
import MapView from 'esri/views/MapView';
import SceneView from 'esri/views/SceneView';

export default Ember.Component.extend({
  didInsertElement() {

    // Grab the config
    let config = this.get('config');
    if (!config.map) {
      return;
    }

    // Add the container as this component
    config.container = this.element;

    // Create the view based on the declared class
    let type = config.map.declaredClass;
    if (type === 'esri.Map'){
      type = config.type === '2d' ? 'esri.WebMap' : 'esri.WebScene';
      delete config.type;
    }
    switch (type) {
    case 'esri.WebScene':
      this.view = new SceneView(config);
      break;

    case 'esri.WebMap':
      this.view = new MapView(config);
      break;
    }
  }
});
```

Update the application.hbs:
```
{{esri-map config=model}}
```

### Test
At this point you should be seeing a 3d map centered on Hamburg (Germany).

### Add a layer to the map

The next steps will illustrate how to update component after they have been instantiated.

In the previous steps learned that the entry points in an ember application are 'routes'.
Routes define the data (model) used by the different components for a route.
The object that controls the state of the model, responds to action requests is the controller.
In the previous steps, we did no create a controller but ember did instantiate a default one for us.

We want to take ownership of the controller. So we need to officially define one:

`ember g controller application`

This command will create a new file app/controllers/application.js

Update app/controllers/application.js
```
import Ember from 'ember';
import FeatureLayer from 'esri/layers/FeatureLayer';

export default Ember.Controller.extend({

  map: Ember.computed.alias('model.map'),

  actions: {
    addLayer() {
      let map = this.get('model.map');
      var featureLayer = new FeatureLayer({
        url: "http://services.arcgis.com/V6ZHFr6zdgNZuVG0/arcgis/rest/services/Landscape_Trees/FeatureServer/0"
      });
      map.add(featureLayer);
    }
  }
});
```

Update the app/templates/application.hbs:
```
{{esri-map config=model}}
<button class="add-layer" {{action 'addLayer'}}>Add layer</button>
```

Update the app/routes/application.js
```
import Ember from 'ember';
import Map from 'esri/Map';

export default Ember.Route.extend({

  model() {
    var map = new Map({
      basemap: "hybrid"
    });

    return {
      type: '2d',
      map: map,
      extent: {
        xmin: -9177811,
        ymin: 4247000,
        xmax: -9176791,
        ymax: 4247784,
        spatialReference: 102100
      }
    };
  }
});
```

We basically added a button on top of the map 'Add Layer'.
This button will trigger an action in the controller.
The action in the controller will get the map from the model and add a layer to it.

### Test

If everything went right, you should see a map on the Warren Wilson College.
If you click the 'Add Layer' button, the map should refresh and a new layer should be visible (orange circles).

## Conclusion
We strongly recommend that you read:
* the guides and the api reference for [ember.js](http://emberjs.com/)
* the document and tutorial for [ember-cli](http://ember-cli.com/)
