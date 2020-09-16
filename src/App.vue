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
          <v-btn-toggle
            tile
            group
            >
            <v-row>
            <v-tooltip top>
              <template v-slot:activator="{ on, attrs }">
                <v-btn @click="canPlaceMarker = !canPlaceMarker; canMoveMarker = false; objectToMove = null"
                  outlined
                  color="grey darken-4"
                  v-on="on"
                  v-bind="attrs"
                  class="ml-3"
                  >
                  Place Marker Mode
                </v-btn>
              </template>
            <span>Select this mode, then click on the map to place a custom marker</span>
            </v-tooltip>
            <v-tooltip bottom>
              <template v-slot:activator="{ on, attrs }">
            <v-btn @click="canPlaceMarker = false; canMoveMarker = !canMoveMarker"
              outlined
              color="grey darken-4"
              v-on="on"
              v-bind="attrs"
              class="ml-3"
              >
              Move Marker Mode
            </v-btn>
              </template>
            <span>Select this mode, click on a custom marker, and then click on the map to move it</span>
            </v-tooltip>
            </v-row>
          </v-btn-toggle>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app color="primary" dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <div class="d-flex align-center">
        <a
          href="https://www.clostra.com/"
        >
        <v-img
          alt="Clostra Logo"
          class="shrink mr-2"
          contain
          src="@/assets/Clostra+Logo+White.png"
          transition="scale-transition"
          width="160"
        />

        </a>
      </div>

      <v-spacer></v-spacer>

        <a
          href="https://worldwind.arc.nasa.gov/"
        >
        <v-img
          alt="NASA Logo"
          class="shrink mr-2 nasa-logo"
          contain
          src="@/assets/918px-NASA_logo.svg"
          transition="scale-transition"
          width="70"
        />
        </a>
          <div>NASA WorldWind </div>
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
          class="pl-6 pr-12"
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
      <v-container class="flex-grow-1">
        <canvas id="canvasOne" class="flex-grow-1">
          Your browser does not support HTML5 Canvas.
        </canvas>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import WorldWind from "@nasaworldwind/worldwind";
import targetLocationsJson from "@/assets/target.json";
import walmartLocationsJson from "@/assets/walmart.json";

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
    canMoveMarker: false,
    showEditAnnotationDialog: false,
    pickedObject: null, // Store a reference to the last pickedObject
    annotationText: "",
    customMarkerCounter: 0,
    objectToMove: null
  }),

  methods: {
    triggerWalmartLayer () {
      this.walmartLocationsPlacemarkLayer.enabled = !this.walmartLocationsPlacemarkLayer.enabled;
      this.wwd.redraw();
      this.activeWalmartLayer = this.walmartLocationsPlacemarkLayer.enabled;
    },

    triggerTargetLayer () {
      this.targetLocationsPlacemarkLayer.enabled = !this.targetLocationsPlacemarkLayer.enabled;
      this.wwd.redraw();
      this.activeTargetLayer = this.targetLocationsPlacemarkLayer.enabled;
    },

    setAnnotationAttributes () {
       // Set default annotation attributes.
        const annotationAttributes = new WorldWind.AnnotationAttributes(null);
        annotationAttributes.cornerRadius = 14;
        annotationAttributes.backgroundColor = WorldWind.Color.MEDIUM_GRAY;
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
        placemarkAttributes.imageSource = "https://files.worldwind.arc.nasa.gov/artifactory/web/0.9.0/images/pushpins/plain-red.png"
        //placemarkAttributes.imageSource = WorldWind.WWUtil.currentUrlSansFilePart() + "./assets/pushpins/plain-red.png"
        console.log('source ' + placemarkAttributes.imageSource)
      } else if (store === "walmart") {
        placemarkAttributes.labelAttributes.color = WorldWind.Color.BLUE;
        placemarkAttributes.imageSource = "https://files.worldwind.arc.nasa.gov/artifactory/web/0.9.0/images/pushpins/plain-blue.png"
      } else if (store === "user") {
        placemarkAttributes.labelAttributes.color = WorldWind.Color.WHITE;
      }

      placemarkAttributes.labelAttributes.offset = new WorldWind.Offset(
        WorldWind.OFFSET_FRACTION, 0.5,
        WorldWind.OFFSET_FRACTION, 1.0);

      placemarkAttributes.imageScale = .4; //Fixme

      return placemarkAttributes;
    },

    addTargetLocations () {
      const placemarkAttributes = this.defineLocationPlacemarkAttributes("target")

      const self = this;
      targetLocationsJson.forEach(function(store) {
        var position = new WorldWind.Position(store.["Address.Latitude"], store["Address.Longitude"], 100.0);
        var placemark = new WorldWind.Placemark(position, false, placemarkAttributes);
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
        placemark.alwaysOnTop = true;

        self.walmartLocationsPlacemarkLayer.addRenderable(placemark);
      });
    }
  },

  mounted () {
    // Add Layers
    this.wwd = new WorldWind.WorldWindow("canvasOne");
    //this.wwd.addLayer(new WorldWind.BMNGOneImageLayer());
    this.wwd.addLayer(new WorldWind.BMNGLandsatLayer());

    //Target Stores Placemark Layer
    this.targetLocationsPlacemarkLayer = new WorldWind.RenderableLayer();
    this.wwd.addLayer(this.targetLocationsPlacemarkLayer);
    this.triggerTargetLayer(); // start with layer off

    //Walmart Stores Placemark Layer
    this.walmartLocationsPlacemarkLayer = new WorldWind.RenderableLayer();
    this.wwd.addLayer(this.walmartLocationsPlacemarkLayer);
    this.triggerWalmartLayer(); // start with layer off

    //User Placemark Layer
    this.userPlacemarkLayer = new WorldWind.RenderableLayer();
    this.wwd.addLayer(this.userPlacemarkLayer);

    this.addTargetLocations();
    this.addWalmartLocations();

    // Set mouse event listener to handle dynamic user placemarks
    // Borrowed from GoToLocation.js -> https://github.com/NASAWorldWind/WebWorldWind/blob/develop/examples/GoToLocation.js
    const self = this;
    new WorldWind.ClickRecognizer(this.wwd, function(e) {
      // Obtain the event location.
      const x = e.clientX,
        y = e.clientY;

      if (self.canMoveMarker && !self.canPlaceMarker) {
        const pickList = self.wwd.pick(self.wwd.canvasCoordinates(x, y));

        if (!self.objectToMove) { // pick up object on first click
          if (!pickList.hasNonTerrainObjects()) return;
          self.objectToMove = pickList.topPickedObject().userObject;

        // If only one thing is picked and it is the terrain, change the object location to that terrain position
        } else if (pickList.objects.length === 1 && pickList.objects[0].isTerrain && self.objectToMove) {
          self.objectToMove.position = pickList.objects[0].position;
          self.objectToMove = null; // reset objectToMove
          self.wwd.redraw()
        }

      }

      if (self.canPlaceMarker && !self.canMoveMarker) {

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
      }

      if (!self.canPlaceMarker && !self.canMoveMarker) {

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
<style scoped>
/* Avoid overlap of nav drawer and app-bar.  Source: https://codepen.io/hamedbaatour/pen/gOpwwjJ */
header {
  postion: absolute !important;
  top:0 !important;
  left: 0 !important;
  z-index: 99999 !important;
}

nav, main{
  margin-top: 4rem !important;
}


main {
  padding: 2rem;
}

/*
.v-main {
  max-height: calc(100% - 80px);
  height: calc(100% - 80px);
}
*/

div.container {
  postion: absolute !important;
  height: 100%;
  width: 100%;
  margin: 0px 0px 0px 0px !important;
  padding: 0px 0px 0px 0px !important;
  max-width: none !important;
}

#canvasOne {
  background-color: black;
  width: 100%;
  max-height: 100%;
}

.v-main {
  padding-top: 0px !important;
  padding-right: 0px !important;
}
</style>
