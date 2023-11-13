<template>
    <div id="cesiumContainer">
    </div>
</template>
<script setup>
import * as Cesium from 'cesium';
import { onMounted } from 'vue';
onMounted(()=>{
  createViewer() //三维建筑模型选择沙盒
})

async function createViewer() {
  Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJlZmM2YjM0ZS00NmNmLTRlMjEtOWZiZS1mMjE1NGMzZmRiMzciLCJpZCI6MTc0MzE2LCJpYXQiOjE2OTgzOTIzNjl9.-EkyRy3gPC0V26Lu9isnBUDBsZkId-fHIKJyG714mfQ';
  const viewer = new Cesium.Viewer("cesiumContainer", {
  terrain: Cesium.Terrain.fromWorldTerrain(),
  });
  viewer.scene.globe.depthTestAgainstTerrain = true;
  // Set the initial camera view to look at Manhattan
  const initialPosition = Cesium.Cartesian3.fromDegrees(
    -74.01881302800248,
    40.69114333714821,
    753
  );
  const initialOrientation = new Cesium.HeadingPitchRoll.fromDegrees(
  21.27879878293835,
  -21.34390550872461,
  0.0716951918898415
  );
  viewer.scene.camera.setView({
    destination: initialPosition,
    orientation: initialOrientation,
    endTransform: Cesium.Matrix4.IDENTITY,
  });
  // Load the NYC buildings tileset
  try {
    const tileset = await Cesium.Cesium3DTileset.fromIonAssetId(75343);
    viewer.scene.primitives.add(tileset);
  } catch (error) {
    console.log(`Error loading tileset: ${error}`);
  }
  const nameOverlay = document.createElement("div");
viewer.container.appendChild(nameOverlay);
nameOverlay.className = "backdrop";
nameOverlay.style.display = "none";
nameOverlay.style.position = "absolute";
nameOverlay.style.bottom = "0";
nameOverlay.style.left = "0";
nameOverlay.style["pointer-events"] = "none";
nameOverlay.style.padding = "4px";
nameOverlay.style.backgroundColor = "black";
//用于保留选中模型
const selected = {
  feature: undefined,
  originalColor: new Cesium.Color()
}
//创建一个实体，用来保存选择实体的相关信息
const selectedEntity = new Cesium.Entity();
//获取左键单击时未选取要素时的默认左键单击处理程序
const clickHandler = viewer.screenSpaceEventHandler.getInputAction(
  Cesium.ScreenSpaceEventType.LEFT_CLICK
);
//更新给定选取功能的“nameOverlay”，在给定的（屏幕）位置。其实就是悬浮时展示的信息框进行新的定位和信息显示
function updateNameOverlay(pickedFeature, position) {
  if (!Cesium.defined(pickedFeature)) {
    nameOverlay.style.display = "none";
    return;
  }
  // A feature was picked, so show its overlay content
  nameOverlay.style.display = "block";
  nameOverlay.style.bottom = `${
    viewer.canvas.clientHeight - position.y
  }px`;
  nameOverlay.style.left = `${position.x}px`;
  //这里的BIN就是提到的悬浮时展示的相关信息
  const name = pickedFeature.getProperty("BIN");
  nameOverlay.textContent = name;
}
//之前提到用实体保存相关信息，此处初始了descripiton后面会进行添加到实体中，也就是const entity = viewer.entities.add({description:''})
function createPickedFeatureDescription(pickedFeature) {
  const description =
    `${
      '<table class="cesium-infoBox-defaultTable"><tbody>' +
      "<tr><th>BIN</th><td>"
    }${pickedFeature.getProperty("BIN")}</td></tr>` +
    `<tr><th>DOITT ID</th><td>${pickedFeature.getProperty(
      "DOITT_ID"
    )}</td></tr>` +
    `<tr><th>SOURCE ID</th><td>${pickedFeature.getProperty(
      "SOURCE_ID"
    )}</td></tr>` +
    `<tr><th>Longitude</th><td>${pickedFeature.getProperty(
      "Longitude"
    )}</td></tr>` +
    `<tr><th>Latitude</th><td>${pickedFeature.getProperty(
      "Latitude"
    )}</td></tr>` +
    `<tr><th>Height</th><td>${pickedFeature.getProperty(
      "Height"
    )}</td></tr>` +
    `<tr><th>Terrain Height (Ellipsoid)</th><td>${pickedFeature.getProperty(
      "TerrainHeight"
    )}</td></tr>` +
    `</tbody></table>`;
  return description;
}
//如果支持剪影，则剪影在鼠标悬停时显示为蓝色，在鼠标单击时显示剪影为绿色。如果不支持剪影，请在鼠标悬停时将要素颜色更改为黄色，在鼠标单击时将要素颜色更改为绿色
if (
  Cesium.PostProcessStageLibrary.isSilhouetteSupported(viewer.scene)
) {
  // Silhouettes are supported 这里建立的是蓝色轮廓模型，用于悬浮
  const silhouetteBlue = Cesium.PostProcessStageLibrary.createEdgeDetectionStage();
  silhouetteBlue.uniforms.color = Cesium.Color.BLUE;
  silhouetteBlue.uniforms.length = 0.01;
  silhouetteBlue.selected = [];
  //这里建立绿色轮廓模型，用于点击后的效果
  const silhouetteGreen = Cesium.PostProcessStageLibrary.createEdgeDetectionStage();
  silhouetteGreen.uniforms.color = Cesium.Color.LIME;
  silhouetteGreen.uniforms.length = 0.01;
  silhouetteGreen.selected = [];

  //将两者轮廓模型进行组合，在实际应用中，可能存在多种不同的轮廓特效，每个特效用于不同的交互或显示需求。通过将这些特效组合到一个阶段中，可以确保它们在渲染过程
  //中按照指定的顺序应用，而不会相互覆盖或产生不可预测的效果。
  //例如，在这个代码中，蓝色轮廓特效用于悬浮时的效果，绿色轮廓特效用于点击后的效果。如果这两个效果分别添加到后期处理阶段，它们可能会相互干扰，导致不期望的渲染
  //结果。通过将它们组合在一起，可以确保在同一阶段中按照指定的顺序应用，从而达到预期的视觉效果。
  //这里将组合后的轮廓模型加到viewer对象的scene属性中的postProcessStages中，以便在渲染场景时应用这些轮廓特效。
  //postProcessStages这个用于在渲染场景后添加后期处理效果的函数，也就是说这个例子里先加载了纽约的模型，这算渲染完毕，点击和悬浮的轮廓模型是后期添加的处理效果
  viewer.scene.postProcessStages.add(
    Cesium.PostProcessStageLibrary.createSilhouetteStage([
      silhouetteBlue,
      silhouetteGreen,
    ])
  );

  //悬浮绑定事件（蓝色）
  viewer.screenSpaceEventHandler.setInputAction(function onMouseMove(
    movement
  ) {
    //如果之前有悬浮高亮显示则去除悬浮高亮显示的要素
    silhouetteBlue.selected = [];
    
    //移动时碰到新的的要素，则进行实时选取，如果没有悬浮到身上，则定义为undefiend
    const pickedFeature = viewer.scene.pick(movement.endPosition);
    //之前提到的，悬浮时的信息框的更新
    updateNameOverlay(pickedFeature, movement.endPosition);

    //如果鼠标悬浮在没有元素的地方则返回。
    if (!Cesium.defined(pickedFeature)) {
      return;
    }

    // Highlight the feature if it's not already selected.
    if (pickedFeature !== selected.feature) {
      silhouetteBlue.selected = [pickedFeature];
    }
  },
  Cesium.ScreenSpaceEventType.MOUSE_MOVE);

  // Silhouette a feature on selection and show metadata in the InfoBox.单击点击事件
  viewer.screenSpaceEventHandler.setInputAction(function onLeftClick(
    movement
  ) {
    // If a feature was previously selected, undo the highlight，如果之前有点击，则去除之前点击的效果
    silhouetteGreen.selected = [];

    // Pick a new feature
    const pickedFeature = viewer.scene.pick(movement.position);
    //如果点击空白地区则就是普通的点击事件，clickHandler()初始化的就是普通点击事件
    if (!Cesium.defined(pickedFeature)) {
      clickHandler(movement);
      return;
    }

    // Select the feature if it's not already selected，当选择了同一个要素时，不继续执行后续代码
    if (silhouetteGreen.selected[0] === pickedFeature) {
      return;
    }

    // Save the selected feature's original color，悬浮时的颜色状态存起来，如果当点击和悬浮重复时，清空悬浮状态的蓝色
    const highlightedFeature = silhouetteBlue.selected[0];
    if (pickedFeature === highlightedFeature) {
      silhouetteBlue.selected = [];
    }

    // Highlight newly selected feature，清空悬浮的蓝色后，装入绿色轮廓
    silhouetteGreen.selected = [pickedFeature];

    // Set feature infobox description
    viewer.selectedEntity = selectedEntity;//selectedEntity用于跟踪现在选的实体
    //将添加description的pickedFeature中选出description重新赋予到新实体selectedEntity中
    selectedEntity.description = createPickedFeatureDescription(
      pickedFeature
    );
  },
  Cesium.ScreenSpaceEventType.LEFT_CLICK);
}
}
</script>
<style>
  #cesiumContainer{
    height: 100%;
    width: 100%;
    margin: 0;
    padding: 0;
  }
</style>