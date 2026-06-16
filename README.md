<!DOCTYPE html>
<html>
  <head>
    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
  </head>
  <body>
    <a-scene webxr="optionalFeatures: hit-test, local-floor; referenceSpaceType: local-floor">
      
      <a-assets>
        <a-asset-item id="fishing-hole" src="models/fishing_hole.glb"></a-asset-item>
      </a-assets>

      <a-entity gltf-model="#fishing-hole" position="1.0 0 -1.5" scale="1 1 1"></a-entity>
      
      <a-entity light="type: ambient; intensity: 0.5;"></a-entity>
      <a-entity light="type: directional; intensity: 0.8;" position="1 4 3"></a-entity>
      
    </a-scene>
  </body>
</html>
