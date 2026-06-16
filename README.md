<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Desk Fishing Hole</title>
    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
    
    <script>
      // Custom fishing logic component
      AFRAME.registerComponent('fishing-game', {
        init: function () {
          this.state = 'idle'; // States: idle, casted, bite, caught
          this.bobber = document.querySelector('#bobber');
          this.fishText = document.querySelector('#ui-text');
          this.hole = document.querySelector('#fishing-hole');
          
          // Listen for trigger presses on both Quest controllers
          this.el.addEventListener('triggerdown', this.handleTrigger.bind(this));
        },

        handleTrigger: function (evt) {
          if (this.state === 'idle') {
            this.castLine();
          } else if (this.state === 'casted') {
            this.reelEarly();
          } else if (this.state === 'bite') {
            this.catchFish();
          } else if (this.state === 'caught') {
            this.resetToIdle();
          }
        },

        castLine: function () {
          this.state = 'casted';
          this.fishText.setAttribute('value', 'Line casted... waiting for a bite.');
          
          // Position bobber in the center of your fishing hole model
          // Adjust these coordinates to match the water level of your GLB
          this.bobber.setAttribute('position', '1.0 0.05 -1.5'); 
          this.bobber.setAttribute('visible', 'true');

          // Random bite timer between 4 to 8 seconds
          let biteDelay = Math.random() * 4000 + 4000;
          this.biteTimeout = setTimeout(() => {
            this.triggerBite();
          }, biteDelay);
        },

        triggerBite: function () {
          this.state = 'bite';
          this.fishText.setAttribute('value', '!! BITE !! REEL IT IN NOW!');
          
          // Quick scale animation to make the bobber "dip" into the water
          this.bobber.setAttribute('animation', 'property: position; to: 1.0 -0.1 -1.5; dur: 200; loops: 5; dir: alternate');
          
          // Give the player a 2-second window to react before the fish escapes
          this.escapeTimeout = setTimeout(() => {
            this.fishEscape();
          }, 2000);
        },

        catchFish: function () {
          clearTimeout(this.escapeTimeout);
          this.bobber.removeAttribute('animation');
          this.state = 'caught';
          
          const fishTypes = ['Bass', 'Trout', 'Salmon', 'Golden Carp', 'Old Boot'];
          const randomFish = fishTypes[Math.floor(Math.random() * fishTypes.length)];
          
          this.fishText.setAttribute('value', `Caught: ${randomFish}! Click to cast again.`);
          this.bobber.setAttribute('visible', 'false');
        },

        reelEarly: function () {
          clearTimeout(this.biteTimeout);
          this.state = 'idle';
          this.fishText.setAttribute('value', 'Reeled in too early. Try again.');
          this.bobber.setAttribute('visible', 'false');
        },

        fishEscape: function () {
          this.bobber.removeAttribute('animation');
          this.state = 'idle';
          this.fishText.setAttribute('value', 'The fish got away! Click to recast.');
          this.bobber.setAttribute('visible', 'false');
        },

        resetToIdle: function () {
          this.state = 'idle';
          this.fishText.setAttribute('value', 'Click trigger to cast into the hole.');
        }
      });
    </script>
  </head>
  <body>

    <!-- WebXR scene with background passthrough enabled -->
    <a-scene webxr="optionalFeatures: hit-test, local-floor; referenceSpaceType: local-floor">
      
      <a-assets>
        <!-- Load your custom Blender environment hole -->
        <a-asset-item id="fishing-hole-glb" src="./models/fishing_hole.glb"></a-asset-item>
      </a-assets>

      <!-- Setup Quest 3 Tracked Controllers and attach our fishing engine -->
      <a-entity id="leftHand" laser-controls="hand: left" fishing-game></a-entity>
      <a-entity id="rightHand" laser-controls="hand: right" fishing-game></a-entity>

      <!-- Your custom 3D model positioned safely to the side of your desk -->
      <a-entity id="fishing-hole" gltf-model="#fishing-hole-glb" position="1.0 0 -1.5" scale="1 1 1"></a-entity>

      <!-- The Bobber: A simple red sphere that appears on cast -->
      <a-sphere id="bobber" radius="0.04" color="#FF0000" position="1.0 0 -1.5" visible="false"></a-sphere>

      <!-- Floating text display floating right over the hole to track states -->
      <a-text id="ui-text" value="Click trigger to cast into the hole." 
              position="1.0 0.8 -1.5" scale="0.6 0.6 0.6" color="#FFFFFF" align="center"></a-text>

      <!-- Ambient environment lighting -->
      <a-entity light="type: ambient; intensity: 0.6;"></a-entity>
      <a-entity light="type: directional; intensity: 0.5;" position="2 4 1"></a-entity>

    </a-scene>
  </body>
</html>
