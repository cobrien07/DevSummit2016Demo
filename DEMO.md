## For a new project
### Step 1

`ember install ember-cli-amd`

### Step 2

In ember-cli-build.js, add the following block inside `new EmberApp`

```
amd :{
  loader: 'https://jsdev.arcgis.com/4.0/',
  packages: [
    'esri','dojo','dojox','dijit',
    'put-selector','xstyle','dbind','dgrid'
  ]
}
```
### Step 3

In app/index.html, add the cdn style after `{{content-for "head"}}`
```
<link rel="stylesheet" href="//jsdev.arcgis.com/4.0/esri/css/main.css">
```

## Part 1: Creating a map component
### Create the component
`ember g component esri-map`
### Update the application.hbs
```
<table-container class="full">
  <table-container-row>
    <h2 id="title">Welcome to Ember</h2>
  </table-container-row>
  <table-container-row class="full-height">
    <div class="full">
      {{! Add esri map component here}}
    </div>
  </table-container-row>
</table-container>
```
## Include the esri map component in the application template
`{{esri-map}}`

## Part 2: Improve the map component
### Add a 2d/3d component property
Goal: Control the type of map (2d, 3d)

New import `import MapView from 'esri/views/MapView';`

New code in didInsertElement:
```
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
```

### Add config support
Goal: control the type the map (2d/3d) and the various properties from the route by using a model.

Create new application route `ember g route application`

application.js
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

Update esri-map.js:
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

    default:

    }
  }
});
```

Update the application.hbs:
```
<table-container class="full">
  <table-container-row>
    <h2 id="title">Welcome to Ember</h2>
  </table-container-row>
  <table-container-row class="full-height">
    <div class="full">
      {{esri-map config=model}}
    </div>
  </table-container-row>
</table-container>
```

### Add a layer to the map
Create the application controller
`ember g controller application`

Update controllers/applicaiton.js
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

Update the application.hbs:
```
{{esri-map config=model mapView=mapView}}
<button class="add-layer" {{action 'addLayer'}}>Add layer</button>
```

Update the routes/application.js
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
