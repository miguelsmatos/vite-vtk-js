<template>
  <div id="vtk-container" ref="vtkContainer"></div>
</template>


<script lang="ts">
import { ref, onMounted } from 'vue';

import '@kitware/vtk.js/Rendering/Profiles/Geometry';

import vtkFullScreenRenderWindow from '@kitware/vtk.js/Rendering/Misc/FullScreenRenderWindow';

import vtkActor from '@kitware/vtk.js/Rendering/Core/Actor';
import vtkMapper from '@kitware/vtk.js/Rendering/Core/Mapper';
import vtkConeSource from '@kitware/vtk.js/Filters/Sources/ConeSource';

import vtkAnnotatedCubeActor from '@kitware/vtk.js/Rendering/Core/AnnotatedCubeActor'
import vtkOrientationMarkerWidget from '@kitware/vtk.js/Interaction/Widgets/OrientationMarkerWidget'
// @ts-expect-error(ts7016)
import vtkInteractiveOrientationWidget from '@kitware/vtk.js/Widgets/Widgets3D/InteractiveOrientationWidget'
// @ts-expect-error(ts7016)
import vtkWidgetManager from '@kitware/vtk.js/Widgets/Core/WidgetManager'

export default {
  name: 'HelloWorld',

  setup() {
    const vtkContainer = ref(null);

    onMounted(() => {

      const fullScreenRenderer = vtkFullScreenRenderWindow.newInstance({
        container: vtkContainer.value! as HTMLElement,
      });
      const coneSource = vtkConeSource.newInstance({ height: 1.0 });
      const mapper = vtkMapper.newInstance();
      mapper.setInputConnection(coneSource.getOutputPort());
      const actor = vtkActor.newInstance();
      actor.setMapper(mapper);
      const renderer = fullScreenRenderer.getRenderer();
      const renderWindow = fullScreenRenderer.getRenderWindow();
      renderer.addActor(actor);
      renderer.resetCamera();
      renderWindow.render();

      // annotated cube widget
      const cube = vtkAnnotatedCubeActor.newInstance()
      const widget = vtkInteractiveOrientationWidget.newInstance()
      widget.placeWidget(cube.getBounds())
      widget.setBounds(cube.getBounds().map((v) => v * 1.025))
      widget.setPlaceFactor(1)
      const interactor = fullScreenRenderer.getRenderWindow().getInteractor();
      const orientationWidget = vtkOrientationMarkerWidget.newInstance({
        actor: cube, interactor: interactor
      });
      cube.setDefaultStyle({

        faceRotation: 0,
        edgeThickness: 0.1,
        resolution: 400
      });
      orientationWidget.setEnabled(true)
      const widgetManager = vtkWidgetManager.newInstance({});
      widgetManager.setRenderer(orientationWidget.getRenderer());
      widgetManager.addWidget(widget);
      widgetManager.enablePicking() // <---
      renderWindow.render()

    });

    return {
      vtkContainer,
    };
  }
}
</script>


<style scoped>
#vtk-container {
  position: absolute;
  margin: 12px;
  top: 30%;
  left: 0;
  right: 0;
  bottom: 0;
  border: 1px solid red;
}
</style>
