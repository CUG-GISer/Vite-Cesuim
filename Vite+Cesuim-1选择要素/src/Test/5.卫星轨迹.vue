<template>
  <div id="cesiumContainer" class="fullSize"></div>
    <div id="loadingOverlay"><h1>Loading...</h1></div>
    <div id="toolbar">
      <div id="propertiesMenu"></div>
    </div>
</template>
  
<script setup>
import * as Cesium from 'cesium';
import { onMounted } from 'vue';
import '../assets/Apps/Sandcastle/Sandcastle-header'
onMounted(() => {
  createViewer1()//3D Model Coloring沙盒
})

async function createViewer1() {
  const viewer = new Cesium.Viewer("cesiumContainer", {
  shouldAnimate: true,
});

const modelCzml = [
  {
    id: "document",
    name: "CZML Model",
    version: "1.0",
  },
  {
    id: "truck model",
    name: "Cesium Milk Truck",
    position: {
      cartographicDegrees: [-77, 37, 0],
    },
    model: {
      gltf:
        "http://192.168.10.11:8888/cesium/gltf/CesiumMilkTruck.glb",
    },
  },
];

function downloadBlob(filename, blob) {
  if (window.navigator.msSaveOrOpenBlob) {
    window.navigator.msSaveBlob(blob, filename);
  } else {
    const elem = window.document.createElement("a");
    elem.href = window.URL.createObjectURL(blob);
    elem.download = filename;
    document.body.appendChild(elem);
    elem.click();
    document.body.removeChild(elem);
  }
}

function reset() {
  viewer.trackedEntity = undefined;
  viewer.dataSources.removeAll();
}

// We fetch the resources that will be embedded in the KMZ file.
const daeModelPromise = Cesium.Resource.fetchBlob({
  url: "http://192.168.10.11:8888/cesium/gltf/CesiumMilkTruck.dae",
});

const texturePromise = Cesium.Resource.fetchBlob({
  url: "http://192.168.10.11:8888/cesium/gltf/CesiumMilkTruck.jpg",
});

// This callback allows us to set the URL of the model to use
// a COLLADA version of the model instead of the glTF version.
// It also lets us specify the files that will be embedded in the exported KMZ.
function modelCallback(modelGraphics, time, externalFiles) {
  const resource = modelGraphics.uri.getValue(time);

  if (resource.url.indexOf("CesiumMilkTruck") !== -1) {
    externalFiles["model/CesiumMilkTruck.dae"] = daeModelPromise;
    externalFiles["model/CesiumMilkTruck.jpg"] = texturePromise;

    return "model/CesiumMilkTruck.dae";
  }

  throw Cesium.RuntimeError("Unknown Model");
}

let filenameToSave = "";
let dataSourcePromise;

Sandcastle.addToolbarMenu(
  [
    {
      text: "Satellites",
      onselect: function () {
        reset();
        filenameToSave = "Satellites.kmz";
        dataSourcePromise = Cesium.CzmlDataSource.load(
          "http://192.168.10.11:8888/cesium/gltf/simple.czml"
        );
        viewer.dataSources.add(dataSourcePromise);

        viewer.camera.flyHome(0);
      },
    },
    {
      text: "Model",
      onselect: function () {
        reset();
        filenameToSave = "Model.kmz";
        dataSourcePromise = Cesium.CzmlDataSource.load(modelCzml);
        viewer.dataSources
          .add(dataSourcePromise)
          .then(function (dataSource) {
            viewer.trackedEntity = dataSource.entities.getById(
              "truck model"
            );
          });
      },
    },
  ],
  "propertiesMenu"
);

Sandcastle.addToolbarButton("Download", function () {
  dataSourcePromise
    .then(function (dataSource) {
      return Cesium.exportKml({
        entities: dataSource.entities,
        kmz: true,
        modelCallback: modelCallback,
      });
    })
    .then(function (result) {
      downloadBlob(filenameToSave, result.kmz);
    })
    .catch(console.error);
});
}

</script>
  
<style scoped>
@import url(../assets/Apps/Sandcastle/templates/bucketRaw.css);

#toolbar {
  background: rgba(42, 42, 42, 0.8);
  padding: 4px;
  border-radius: 4px;
}

#toolbar input {
  vertical-align: middle;
  padding-top: 2px;
  padding-bottom: 2px;
}

#toolbar .header {
  font-weight: bold;
}
</style>