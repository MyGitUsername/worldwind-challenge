<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      app
      >
      <v-list dense>
        <v-list-item>
           <v-switch
              v-model="activeWalmartLayer"
              label="Walmart"
              color="blue"
              v-on:click="triggerWalmartLayer()"
              hide-details
            ></v-switch>
        </v-list-item>
        <v-list-item>
           <v-switch
              v-model="activeTargetLayer"
              label="Target"
              color="red"
              v-on:click="triggerTargetLayer()"
              hide-details
            ></v-switch>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app color="primary" dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <div class="d-flex align-center">
        <v-img
          alt="Clostra Logo"
          class="shrink mr-2"
          contain
          src="@/assets/Clostra+logo.png"
          transition="scale-transition"
          width="40"
        />

        <v-btn
          href="https://www.clostra.com/"
          target="_blank"
          text
        >
          <span class="mr-2">Clostra</span>
        </v-btn>
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
        text
      >
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container>
        <canvas id="canvasOne" width="1024" height="768">
          Your browser does not support HTML5 Canvas.
        </canvas>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import WorldWind from "@nasaworldwind/worldwind";
import targetLocationsJson from "@/assets/target-locations.json";
import walmartLocationsJson from "@/assets/walmart-locations.json";

export default {
  name: "WorldWiind",

  data: () => ({
    drawer: null,
    wwd: null,
    targetLocationsPlacemarkLayer: null,
    walmartLocationsPlacemarkLayer: null,
    userPlacemarkLayer: null,
    activeTargetLayer: true,
    activeWalmartLayer: true
  }),

  methods: {
    triggerWalmartLayer () {
      console.log('activeWalmartLayer is ' + this.activeWalmartLayer)
      if (!this.activeWalmartLayer) {
        this.removeWalmartLocations();
      } else {
        this.wwd.addLayer(this.walmartLocationsPlacemarkLayer);
      }

    },

    triggerTargetLayer () {
      if (!this.activeTargetLayer) {
        this.removeTargetLocations();
      } else {
        this.wwd.addLayer(this.targetLocationsPlacemarkLayer);
      }
    },

    defineLocationPlacemarkAttributes (store) {
      const placemarkAttributes = new WorldWind.PlacemarkAttributes(null);

      placemarkAttributes.imageOffset = new WorldWind.Offset(
        WorldWind.OFFSET_FRACTION, 0.3,
        WorldWind.OFFSET_FRACTION, 0.0);

      if (store === "target") {
        placemarkAttributes.labelAttributes.color = WorldWind.Color.RED;
      } else if (store === "walmart") {
        placemarkAttributes.labelAttributes.color = WorldWind.Color.BLUE;
      }
      placemarkAttributes.labelAttributes.offset = new WorldWind.Offset(
        WorldWind.OFFSET_FRACTION, 0.5,
        WorldWind.OFFSET_FRACTION, 1.0);

      placemarkAttributes.imageSource = '../assets/push-pin-red.png';

      return placemarkAttributes;
    },

    addTargetLocations () {
      const placemarkAttributes = this.defineLocationPlacemarkAttributes("target")

      const self = this;
      targetLocationsJson.forEach(function(store) {
        var position = new WorldWind.Position(store.latitude, store.longitude, 100.0);
        var placemark = new WorldWind.Placemark(position, false, placemarkAttributes);

        placemark.label = "Target\n" +
          "Lat " + placemark.position.latitude.toPrecision(4).toString() + "\n" +
          "Lon " + placemark.position.longitude.toPrecision(5).toString();
        placemark.alwaysOnTop = true;

        self.targetLocationsPlacemarkLayer.addRenderable(placemark);
      });
    },

    addWalmartLocations () {
      const placemarkAttributes = this.defineLocationPlacemarkAttributes("walmart")

      const self = this;
      walmartLocationsJson.forEach(function(store) {
        var position = new WorldWind.Position(store.latitude, store.longitude, 100.0);
        var placemark = new WorldWind.Placemark(position, false, placemarkAttributes);

        placemark.label = "Walmart\n" +
          "Lat " + placemark.position.latitude.toPrecision(4).toString() + "\n" +
          "Lon " + placemark.position.longitude.toPrecision(5).toString();
        placemark.alwaysOnTop = true;

        self.walmartLocationsPlacemarkLayer.addRenderable(placemark);
      });
    },
    removeWalmartLocations () {
      this.wwd.removeLayer(this.walmartLocationsPlacemarkLayer)
    },

    removeTargetLocations () {
      this.wwd.removeLayer(this.targetLocationsPlacemarkLayer)
    }
  },

  mounted () {
    // Add Layers
    this.wwd = new WorldWind.WorldWindow("canvasOne");
    this.wwd.addLayer(new WorldWind.BMNGOneImageLayer());
    this.wwd.addLayer(new WorldWind.BMNGLandsatLayer());

    //Target Stores Placemark Layer
    this.targetLocationsPlacemarkLayer = new WorldWind.RenderableLayer();
    this.wwd.addLayer(this.targetLocationsPlacemarkLayer);

    //Walmart Stores Placemark Layer
    this.walmartLocationsPlacemarkLayer = new WorldWind.RenderableLayer();
    this.wwd.addLayer(this.walmartLocationsPlacemarkLayer);

    //User Placemark Layer
    this.userPlacemarkLayer = new WorldWind.RenderableLayer();
    this.wwd.addLayer(this.userPlacemarkLayer);

    //this.wwd.addLayer(new WorldWind.CompassLayer());
    this.wwd.addLayer(new WorldWind.CoordinatesDisplayLayer(this.wwd));
    //this.wwd.addLayer(new WorldWind.ViewControlsLayer(this.wwd));

    //this.addPlacemarkers();

    this.addTargetLocations();
    this.addWalmartLocations();

    // Set mouse event listener to handle dynamic user placemarks
    // Borrowed from GoToLocation.js -> https://github.com/NASAWorldWind/WebWorldWind/blob/develop/examples/GoToLocation.js
    const self = this;
    this.wwd.addEventListener("click", function(e) {
      // Obtain the event location.
      var x = e.clientX,
        y = e.clientY;

      // Perform the pick. Must first convert from window coordinates to canvas coordinates, which are
      // relative to the upper left corner of the canvas rather than the upper left corner of the page.
      var pickList = self.wwd.pick(self.wwd.canvasCoordinates(x, y));

      // If only one thing is picked and it is the terrain, tell the WorldWindow to go to the picked location.
      if (pickList.objects.length === 1 && pickList.objects[0].isTerrain) {
        var position = pickList.objects[0].position;
        self.wwd.goTo(new WorldWind.Location(position.latitude, position.longitude));
      }
    });
  }
};
</script>
