<template>
  <div id="map" />
</template>
<style>
html,
body {
  margin: 0;
  height: 100%;
}
#c {
  width: 100%;
  height: 100%;
  display: block;
}
</style>
<script src="https://unpkg.com/three@0.132/build/three.min.js"></script>
<script src="https://unpkg.com/three@0.132/examples/js/loaders/GLTFLoader.js"></script>
<script src="https://threejsfundamentals.org/threejs/../3rdparty/dat.gui.module.js"></script>
<script src="https://cdn.jsdelivr.net/gh/jscastro76/threebox@v.2.2.5/dist/threebox.min.js" type="text/javascript"></script>


<script>
import mapboxgl from "mapbox-gl";
import threebox from "threebox-plugin/dist/threebox";

//import * as THREE from 'https://threejsfundamentals.org/threejs/resources/threejs/r132/build/three.module.js';
//import {OrbitControls} from 'https://threejsfundamentals.org/threejs/resources/threejs/r132/examples/jsm/controls/OrbitControls.js';
//import {GUI} from 'https://threejsfundamentals.org/threejs/../3rdparty/dat.gui.module.js';

export default {
  name: "Threebox-project",
  data() {
    return {
      accessToken:
        'pk.eyJ1IjoiZXZhbmRlbGJlY3EiLCJhIjoiY2wyeWxnMnplMDNmbjNqcW80OXo5ZjR4NyJ9.b9tdFTYQ13LuscLgGq0v2A',
    };
  },
  mounted() {
    mapboxgl.accessToken = this.accessToken;

    this.map = new mapboxgl.Map({
      container: "map",
      interactive: true,
      style: 'mapbox://styles/mapbox-map-design/ckhqrf2tz0dt119ny6azh975y',
      zoom: 14.5,
      center: [55.5103, -20.8869],
      pitch: 60,
      bearing: 270,
      antialias: true,
    });
    this.map.on('load', () => {
        this.map.addSource('mapbox-dem', {
            'type': 'raster-dem',
            'url': 'mapbox://mapbox.mapbox-terrain-dem-v1',
            'tileSize': 512,
            'maxzoom': 14
        });
        // add the DEM source as a terrain layer with exaggerated height
        this.map.setTerrain({ 'source': 'mapbox-dem', 'exaggeration': 1.5 });

        // add a sky layer that will show when the map is highly pitched
        this.map.addLayer({
            'id': 'sky',
            'type': 'sky',
            'paint': {
                'sky-type': 'atmosphere',
                'sky-atmosphere-sun': [0.0, 0.0],
                'sky-atmosphere-sun-intensity': 15
            }
        });
    });

    this.map.on("style.load", () => {

      window.tb = new Threebox(
        this.map,
        this.map.getCanvas().getContext("webgl"),
        {
          defaultLights: true,
          //antialias: true,
        }
      );

      let _this = this;
      if (this.map.getLayer("custom_layer") == null) {
        this.map.addLayer(
          {
            id: "custom_layer",
            type: "custom",
            renderingMode: "3d",

            onAdd: function (map, mbxContext) {
              //load hemisphere gltf from russellsresume server
              var options = {
                obj: "https://russellsresume.com/gltf/hemisphere-06.gltf",
                type: "gltf",
                scale: 1,
                units: "meters",
                rotation: { x: 90, y: 0, z: 0 }, //default rotation
              };

              window.tb.loadObj(options, function (model) {
                let hemisphere = model.setCoords([55.5103, -20.8869, 0]);
                hemisphere.setAnchor("center");
                window.tb.add(hemisphere);
              });

              {
                const color = 0xffffff;
                const intensity = 2;
                const light = new THREE.DirectionalLight(color, intensity);
                light.position.set(0, 0, 1000).normalize();
                light.target.position.set(0, 0, 0).normalize();
                window.tb.add(light);
                window.tb.add(light.target);

                const helper = new THREE.DirectionalLightHelper(light);
                window.tb.add(helper);

                function updateLight() {
                  light.target.updateMatrixWorld();
                  helper.update();
                }
                updateLight();

                const gui = new GUI();
                gui
                  .addColor(new ColorGUIHelper(light, "color"), "value")
                  .name("color");
                gui.add(light, "intensity", 0, 2, 0.01);

                makeXYZGUI(gui, light.position, "position", updateLight);
                makeXYZGUI(gui, light.target.position, "target", updateLight);
              }
              function makeXYZGUI(gui, vector3, name, onChangeFn) {
                const folder = gui.addFolder(name);
                folder.add(vector3, "x", -10, 10).onChange(onChangeFn);
                folder.add(vector3, "y", 0, 10).onChange(onChangeFn);
                folder.add(vector3, "z", -10, 10).onChange(onChangeFn);
                folder.open();
              }
            },
            render: function (gl, matrix) {
              window.tb.update();
              _this.animatePulse();
            },
          }
          //'place_label_town'
        );
      }
    });
  },
  methods: {
    sphere(lon, lat) {
      let origin = [lon, lat, 0];
      return window.tb
        .sphere({ color: "red", material: "MeshToonMaterial" })
        .setCoords(origin);
    },
    animatePulse() {
      this.map.triggerRepaint();
    },
  },
};
</script>

<style scoped>
#map {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}
</style>