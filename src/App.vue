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
              color="blue darken-1"
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
        <v-list-item></v-list-item>
        <v-divider></v-divider>
        <v-list-item></v-list-item>
        <v-list-item>
          <v-btn @click="canPlaceMarker = !canPlaceMarker">
            <v-img right src="@/assets/plain-black.png" aspect-ratio="1"></v-img>
            Place Marker
          </v-btn>
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
       <v-dialog
      v-model="showEditAnnotationDialog"
      max-width="290"
    >
      <v-card>
        <v-card-title class="headline">Edit Marker</v-card-title>

          <v-text-field
          v-model="annotationText"
          value="Edit Annotation Text"
        ></v-text-field>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="blue darken-1"
            text
            @click="pickedObject.text = annotationText; showEditAnnotationDialog = false;"
          >
            Submit
          </v-btn>
          <v-btn
            color="blue darken-1"
            text
            @click="showEditAnnotationDialog = false"
          >
            Cancel
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
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
    activeWalmartLayer: true,
    canPlaceMarker: false, // If set to true, click on wwd will add placemarker
    showEditAnnotationDialog: false,
    pickedObject: null, // Store a reference to the last pickedObject
    annotationText: "",
    customMarkerCounter: 0
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

    setAnnotationAttributes () {
       // Set default annotation attributes.
        const annotationAttributes = new WorldWind.AnnotationAttributes(null);
        annotationAttributes.cornerRadius = 14;
        annotationAttributes.backgroundColor = WorldWind.Color.GREEN;
        annotationAttributes.drawLeader = true;
        annotationAttributes.leaderGapWidth = 40;
        annotationAttributes.leaderGapHeight = 30;
        annotationAttributes.opacity = 1;
        annotationAttributes.scale = 1;
        annotationAttributes.width = 200;
        annotationAttributes.height = 100;
        annotationAttributes.textAttributes.color = WorldWind.Color.BLACK;
        annotationAttributes.insets = new WorldWind.Insets(10, 10, 10, 10);

        return annotationAttributes;
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
      } else if (store === "user") {
        placemarkAttributes.labelAttributes.color = WorldWind.Color.WHITE;
      }

      placemarkAttributes.labelAttributes.offset = new WorldWind.Offset(
        WorldWind.OFFSET_FRACTION, 0.5,
        WorldWind.OFFSET_FRACTION, 1.0);

      placemarkAttributes.imageScale = 8; //Fixme
      placemarkAttributes.imageSource = "@/assets/plain-red.png";

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


    this.addTargetLocations();
    this.addWalmartLocations();

    // Set mouse event listener to handle dynamic user placemarks
    // Borrowed from GoToLocation.js -> https://github.com/NASAWorldWind/WebWorldWind/blob/develop/examples/GoToLocation.js
    const self = this;
    this.wwd.addEventListener("mouseup", function(e) {
      if (e.shiftKey) {
          console.log('click and drag')
        // Obtain the event location.
        const x = e.clientX,
          y = e.clientY;
        console.log('on drag end the new coors are ' + x + ', ' + y)
      }
    });

    new WorldWind.DragRecognizer(this.wwd, function() {
      return;
    });

    new WorldWind.ClickRecognizer(this.wwd, function(e) {
      // Obtain the event location.
      const x = e.clientX,
        y = e.clientY;

      if (self.canPlaceMarker) {

        // Perform the pick. Must first convert from window coordinates to canvas coordinates, which are
        // relative to the upper left corner of the canvas rather than the upper left corner of the page.
        const pickList = self.wwd.pick(self.wwd.canvasCoordinates(x, y));

        // If only one thing is picked and it is the terrain, tell the WorldWindow to go to the picked location.
        if (pickList.objects.length === 1 && pickList.objects[0].isTerrain) {
          const position = pickList.objects[0].position;
          //const placemarkAttributes = self.defineLocationPlacemarkAttributes("user")

          const annotationAttributes = self.setAnnotationAttributes();
          const annotation = new WorldWind.Annotation(position, annotationAttributes);
          annotation.displayName = "My Placemark";
          annotation.text = "Marker " + self.customMarkerCounter++;
          self.userPlacemarkLayer.addRenderable(annotation);

          /*
          var placemark = new WorldWind.Placemark(position);
          console.log('placemark is ' + JSON.stringify(placemark))

          placemark.label = "My Placemark\n" +
            "Lat " + placemark.position.latitude.toPrecision(4).toString() + "\n" +
            "Lon " + placemark.position.longitude.toPrecision(5).toString();
          placemark.alwaysOnTop = true;
          self.userPlacemarkLayer.addRenderable(placemark);
          */
        }
      } else {

        // see what the user is picking
        var pickList = self.wwd.pick(self.wwd.canvasCoordinates(x, y));
        if (!pickList.hasNonTerrainObjects()) return;
        console.log('get picklist ' + pickList.topPickedObject().userObject);
        const topPickedObject = pickList.topPickedObject().userObject;
        if (topPickedObject.text !== undefined) {  //FIXME: hacky solution to ignore placemarkers
          self.showEditAnnotationDialog = true;
          self.pickedObject = topPickedObject;
          self.annotationText = topPickedObject.text;
        }
      }

    });
  }
};
</script>
