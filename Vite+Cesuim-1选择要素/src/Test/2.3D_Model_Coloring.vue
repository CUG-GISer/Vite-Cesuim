<template>
<div id="cesiumContainer" class="fullSize"></div>
    <div id="loadingOverlay"><h1>Loading...</h1></div>
    <div id="toolbar">
      <table>
        <tbody>
          <tr>
            <td class="header">Model Color</td>
          </tr>
          <tr>
            <td>Mode</td>
            <td>
              <select data-bind="options: colorBlendModes, value: colorBlendMode"></select>
            </td>
          </tr>
          <tr>
            <td>Color</td>
            <td><select data-bind="options: colors, value: color"></select></td>
          </tr>
          <tr>
            <td>Alpha</td>
            <td>
              <input type="range" min="0.0" max="1.0" step="0.01" data-bind="value: alpha, valueUpdate: 'input'">
              <input type="text" size="5" data-bind="value: alpha">
            </td>
          </tr>
          <tr>
            <td data-bind="style: { color: colorBlendAmountEnabled ? '' : 'gray'}">
              Mix
            </td>
            <td>
              <input type="range" min="0.0" max="1.0" step="0.01" data-bind="value: colorBlendAmount, valueUpdate: 'input', enable: colorBlendAmountEnabled">
              <input type="text" size="5" data-bind="value: colorBlendAmount, enable: colorBlendAmountEnabled">
            </td>
          </tr>
          <tr>
            <td class="header">Model Silhouette</td>
          </tr>
          <tr>
            <td>Color</td>
            <td>
              <select data-bind="options: silhouetteColors, value: silhouetteColor"></select>
            </td>
          </tr>
          <tr>
            <td>Alpha</td>
            <td>
              <input type="range" min="0.0" max="1.0" step="0.01" data-bind="value: silhouetteAlpha, valueUpdate: 'input'">
              <input type="text" size="5" data-bind="value: silhouetteAlpha">
            </td>
          </tr>
          <tr>
            <td>Size</td>
            <td>
              <input type="range" min="0.0" max="10.0" step="0.01" data-bind="value: silhouetteSize, valueUpdate: 'input'">
              <input type="text" size="5" data-bind="value: silhouetteSize">
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
  
  <script setup>
  import * as Cesium from 'cesium';
  import { onMounted } from 'vue';
  import '../assets/Apps/Sandcastle/Sandcastle-header'
  onMounted(()=>{
    createViewer1()//3D Model Coloring沙盒
  })
  
  let createViewer1 = ()=>{
    const viewer = new Cesium.Viewer("cesiumContainer", {
    infoBox: false,
    selectionIndicator: false,
    shadows: true,
    shouldAnimate: true,
  });
  
  let entity;
  
  function getColorBlendMode(colorBlendMode) {
    return Cesium.ColorBlendMode[colorBlendMode.toUpperCase()];
  }
  
  function getColor(colorName, alpha) {
    const color = Cesium.Color[colorName.toUpperCase()];
    return Cesium.Color.fromAlpha(color, parseFloat(alpha));
  }
  
  // The viewModel tracks the state of our mini application.
  const viewModel = {
    color: "Red",
    colors: ["White", "Red", "Green", "Blue", "Yellow", "Gray"],
    alpha: 1.0,
    colorBlendMode: "Highlight",
    colorBlendModes: ["Highlight", "Replace", "Mix"],
    colorBlendAmount: 0.5,
    colorBlendAmountEnabled: false,
    silhouetteColor: "Red",
    silhouetteColors: ["Red", "Green", "Blue", "Yellow", "Gray"],
    silhouetteAlpha: 1.0,
    silhouetteSize: 2.0,
  };
  
  // Convert the viewModel members into knockout observables.
  Cesium.knockout.track(viewModel);
  
  // Bind the viewModel to the DOM elements of the UI that call for it.
  const toolbar = document.getElementById("toolbar");
  Cesium.knockout.applyBindings(viewModel, toolbar);
  
  //konkout的getObservable是监视某个对象的，这里监视了viewModel的color属性，当color属性变换时则订阅(subscribe)函数发生回调function (newValue)
  //其中newValue就是color属性变化的新值，作为一个参数传入getColor()中。
  Cesium.knockout
    .getObservable(viewModel, "color")
    .subscribe(function (newValue) {
      entity.model.color = getColor(newValue, viewModel.alpha);
    });
  
  Cesium.knockout
    .getObservable(viewModel, "alpha")
    .subscribe(function (newValue) {
      entity.model.color = getColor(viewModel.color, newValue);
    });
  
  Cesium.knockout
    .getObservable(viewModel, "colorBlendMode")
    .subscribe(function (newValue) {
      const colorBlendMode = getColorBlendMode(newValue);
      entity.model.colorBlendMode = colorBlendMode;
      viewModel.colorBlendAmountEnabled =
        colorBlendMode === Cesium.ColorBlendMode.MIX;
    });
  
  Cesium.knockout
    .getObservable(viewModel, "colorBlendAmount")
    .subscribe(function (newValue) {
      entity.model.colorBlendAmount = parseFloat(newValue);
    });
  
  Cesium.knockout
    .getObservable(viewModel, "silhouetteColor")
    .subscribe(function (newValue) {
      entity.model.silhouetteColor = getColor(
        newValue,
        viewModel.silhouetteAlpha
      );
    });
  
  Cesium.knockout
    .getObservable(viewModel, "silhouetteAlpha")
    .subscribe(function (newValue) {
      entity.model.silhouetteColor = getColor(
        viewModel.silhouetteColor,
        newValue
      );
    });
  
  Cesium.knockout
    .getObservable(viewModel, "silhouetteSize")
    .subscribe(function (newValue) {
      entity.model.silhouetteSize = parseFloat(newValue);
    });
  
  function createModel(url, height) {
    viewer.entities.removeAll();
  
    const position = Cesium.Cartesian3.fromDegrees(
      -123.0744619,
      44.0503706,
      height
    );
    const heading = Cesium.Math.toRadians(135);
    const pitch = 0;
    const roll = 0;
    const hpr = new Cesium.HeadingPitchRoll(heading, pitch, roll);
    const orientation = Cesium.Transforms.headingPitchRollQuaternion(
      position,
      hpr
    );
  
    entity = viewer.entities.add({
      name: url,
      position: position,
      orientation: orientation,
      model: {
        uri: url,
        minimumPixelSize: 128,
        maximumScale: 20000,
        color: getColor(viewModel.color, viewModel.alpha),
        colorBlendMode: getColorBlendMode(viewModel.colorBlendMode),
        colorBlendAmount: parseFloat(viewModel.colorBlendAmount),
        silhouetteColor: getColor(
          viewModel.silhouetteColor,
          viewModel.silhouetteAlpha
        ),
        silhouetteSize: parseFloat(viewModel.silhouetteSize),
      },
    });
    viewer.trackedEntity = entity;
  }
  
  const options = [
    {
      text: "Aircraft",
      onselect: function () {
        createModel(
          "http://192.168.10.11:8888/cesium/gltf/Cesium_Air.glb",
          5000.0
        );
      },
    },
    {
      text: "Ground Vehicle",
      onselect: function () {
        createModel(
          "http://192.168.10.11:8888/cesium/gltf/GroundVehicle.glb",
          0
        );
      },
    },
    {
      text: "Hot Air Balloon",
      onselect: function () {
        createModel(
          "http://192.168.10.11:8888/cesium/gltf/CesiumBalloon.glb",
          1000.0
        );
      },
    },
    {
      text: "Milk Truck",
      onselect: function () {
        createModel(
          "http://192.168.10.11:8888/cesium/gltf/CesiumMilkTruck.glb",
          0
        );
      },
    },
    {
      text: "Skinned Character",
      onselect: function () {
        createModel(
          "http://192.168.10.11:8888/cesium/gltf/Cesium_Man.glb",
          0
        );
      },
    },
  ];
  
  Sandcastle.addToolbarMenu(options);
  
  Sandcastle.addToggleButton("Shadows", viewer.shadows, function (
    checked
  ) {
    viewer.shadows = checked;
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