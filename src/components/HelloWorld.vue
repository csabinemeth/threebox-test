<template>
  <div id="map"></div>
</template>

<script>
import mapboxgl from "mapbox-gl"; // or "const mapboxgl = require('mapbox-gl');"
import { Threebox } from "threebox-plugin";

export default {
  name: "MainPage",
  props: {
    msg: String,
  },

  data() {
    return {
      coords: [
        { lat: 47.694405066290365, lon: 17.626292940496935, alt: 1 },
        { lat: 47.69442549153422, lon: 17.626251215118973, alt: 1 },
        { lat: 47.69441017260208, lon: 17.62614121184979, alt: 2 },
        { lat: 47.694405066290365, lon: 17.6260501746615, alt: 1 },
        { lat: 47.69439485366541, lon: 17.625981896770288, alt: 2 },
        { lat: 47.69434889682846, lon: 17.625883273149643, alt: 2 },
        { lat: 47.694295280467465, lon: 17.62586810028493, alt: 1 },
        { lat: 47.694285067821035, lon: 17.625852927420212, alt: 2 },
        { lat: 47.6942263450652, lon: 17.625871893501106, alt: 2 },
        { lat: 47.69409613350118, lon: 17.625837754555498, alt: 1 },
        { lat: 47.69397358114387, lon: 17.62591741209525, alt: 2 },
        { lat: 47.693922517576674, lon: 17.625947757824676, alt: 2 },
        { lat: 47.69391485803729, lon: 17.626171557579216, alt: 2 },
        { lat: 47.69394294300958, lon: 17.626304320145472, alt: 1 },
        { lat: 47.69401698513669, lon: 17.626327079442543, alt: 2 },
        { lat: 47.694057835920475, lon: 17.626323286226363, alt: 2 },
        { lat: 47.69411400569588, lon: 17.626327079442543, alt: 1 },
        { lat: 47.69420081341162, lon: 17.626304320145472, alt: 1 },
        { lat: 47.6943335778738, lon: 17.62633845909108, alt: 2 },
        { lat: 47.69436932209435, lon: 17.6263346658749, alt: 2 },
      ],
    };
  },

  mounted() {
    mapboxgl.accessToken =
      "pk.eyJ1IjoiY3NhYmluZW1ldGgiLCJhIjoiY2xlOGxibzV3MGFvdDN2cXNjbnM2Zm83dSJ9.1TJNwxyuw5o0pPxHZlfROQ";
    const map = new mapboxgl.Map({
      container: "map",
      // Choose from Mapbox's core styles, or make your own style with Mapbox Studio
      style: "mapbox://styles/mapbox/light-v11",
      center: { lng: 17.62695, lat: 47.69433 },
      zoom: 15.4,
      pitch: 64.9,
      bearing: 172.5,
      antialias: true, // create the gl context with MSAA antialiasing, so custom layers are antialiased
    });

    // eslint-disable-next-line no-undef
    const tb = (window.tb = new Threebox(
      map,
      map.getCanvas().getContext("webgl"),
      {
        defaultLights: true,
      }
    ));

    map.on("style.load", () => {
      map.addLayer({
        id: "add-3d-buildings",
        source: "composite",
        "source-layer": "building",
        filter: ["==", "extrude", "true"],
        type: "fill-extrusion",
        minzoom: 15,
        paint: {
          "fill-extrusion-color": "#aaa",

          // Use an 'interpolate' expression to
          // add a smooth transition effect to
          // the buildings as the user zooms in.
          "fill-extrusion-height": [
            "interpolate",
            ["linear"],
            ["zoom"],
            15,
            0,
            15.05,
            ["get", "height"],
          ],
          "fill-extrusion-base": [
            "interpolate",
            ["linear"],
            ["zoom"],
            15,
            0,
            15.05,
            ["get", "min_height"],
          ],
          "fill-extrusion-opacity": 0.6,
        },
      });

      for (let i = 0; i <= this.coords.length; i++) {
        console.log(i);
        const modelId = `model_${i}`;
        console.log(modelId);

        map.addLayer({
          id: modelId,
          type: "custom",
          renderingMode: "3d",
          onAdd: () => {
            // Creative Commons License attribution:  Metlife Building model by https://sketchfab.com/NanoRay
            // https://sketchfab.com/3d-models/metlife-building-32d3a4a1810a4d64abb9547bb661f7f3
            const scale = 2.5;
            const options = {
              obj: "orb2.gltf",
              type: "gltf",
              scale: { x: scale, y: scale, z: scale },
              units: "meters",
              rotation: { x: 90, y: -90, z: 0 },
            };

            tb.loadObj(options, (model) => {
              model.setCoords([this.coords[i].lon, this.coords[i].lat]);

              model.setRotation({ x: 0, y: 0, z: 241 });

              model.traverse((node) => {
                if (node.isMesh) {
                  // Modify the color property of the material
                  node.material.color.set("#03fdf4"); // Set to red color
                  node.castShadow = false;
                            node.receiveShadow = false;

                }
              });
              const altitude = this.coords[i].alt; // Replace with the desired altitude (in meters)

              const position = model.position;
              position.set(position.x, position.y, altitude);


              tb.add(model);
            });
          },

          render: function () {
            tb.update();
          },
        });
      }
    });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

#map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
