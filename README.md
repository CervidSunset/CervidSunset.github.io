<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Desk Fishing Hole</title>
    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
    
    <script>
      AFRAME.registerComponent('game-manager', {
        init: function () {
          this.state = 'placing'; 
          
          this.bobber = document.querySelector('#bobber');
          this.fishText = document.querySelector('#ui-text');
          this.hole = document.querySelector('#fishing-hole');
          this.reticle = document.querySelector('#reticle');
          this.startMenu = document.querySelector('#start-menu');
          
          this.targetPos = {x: 0, y: 0, z: 0}; 

          // We still use the controller trigger to confirm the placement and fish
          const rightHand = document.querySelector('#rightHand');
          rightHand.addEventListener('triggerdown', this.handleTrigger.bind(this));
        },

        handleTrigger: function (evt) {
          if (this.state === 'placing') {
            // Only allow placement if the AR hit test has found a real physical surface
            if (this.reticle.getAttribute('visible')) {
              this.placePond();
            }
          } else if (this.state === 'idle') {
            this.castLine();
          } else if (this.state === 'casted') {
            this.reelEarly();
          } else if (this.state === 'bite') {
            this.catchFish();
          } else if (this.state === 'caught') {
            this.resetToIdle();
          }
        },

        placePond: function() {
          this.state = 'idle';
          
          // Grab the exact real-world coordinates from the AR depth sensor
          let arPosition = this.reticle.getAttribute('position');
          this.targetPos = {x: arPosition.x, y: arPosition.y, z: arPosition.z};
          
          // Move the 3D Pond to the physical surface
          this.hole.setAttribute('position', this.targetPos);
          this.hole.setAttribute('visible', 'true');
          
          // Hover the UI text 0.8 meters above the physical floor
          this.fishText.setAttribute('position', {
             x: this.targetPos.x,
             y: this.targetPos.y + 0.8,
             z: this.targetPos.z
          });
          this.fishText.setAttribute('value', 'Pond placed! Click trigger to cast.');
          this.fishText.setAttribute('visible', 'true');

          // Shut off the AR Hit Test engine so the reticle goes away
          this.reticle.removeAttribute('ar-hit-test');
          this.reticle.setAttribute('visible', 'false');
          this.startMenu.setAttribute('visible', 'false');
        },

        castLine: function () {
          this.state = 'casted';
          this.fishText.setAttribute('value', 'Line casted... waiting for a bite.');
          
          // Spawn the bobber flush with the custom water level
          this.bobber.setAttribute('position', {
             x: this.targetPos.x,
             y: this.targetPos.y + 0.05,
             z: this.targetPos.z
          });
          this.bobber.setAttribute('visible', 'true');

          let biteDelay = Math.random() * 4000 + 4000;
          this.biteTimeout = setTimeout(() => {
            this.triggerBite();
          }, biteDelay);
        },

        triggerBite: function () {
          this.state = 'bite';
          this.fishText.setAttribute('value', '!! BITE !! REEL IT IN NOW!');
          
          let dipY = this.targetPos.y - 0.1;
          this.bobber.setAttribute('animation', `property: position; to: ${this.targetPos.x} ${dipY} ${this.targetPos.z}; dur: 200; loops: 5; dir: alternate`);
          
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

    <a-scene 
      webxr="optionalFeatures: hit-test, local-floor; referenceSpaceType: local-floor"
      xr-mode-ui="XRMode: xr"
      background="transparent: true"
      game-manager>
      
      <a-assets>
        <a-asset-item id="fishing-hole-glb" src="./models/fishing_hole.glb"></a-asset-item>
      </a-assets>

      <a-entity id="start-menu" position="0 1.5 -1.5">
        <a-plane color="#000000" opacity="0.8" width="1.5" height="0.6" position="0 0 -0.01"></a-plane>
        <a-text value="Welcome to Desk Fishing!\n\nLook at the real floor to move the ring,\nthen pull the trigger to place your pond." 
                align="center" color="#FFFFFF" width="1.5"></a-text>
      </a-entity>

      <a-entity id="leftHand" laser-controls="hand: left"></a-entity>
      <a-entity id="rightHand" laser-controls="hand: right" ar-hit-test="target: #reticle;"></a-entity>

      <a-entity id="reticle" visible="false">
  <a-ring rotation="-90 0 0" radius-inner="0.15" radius-outer="0.2" color="#00FF00"></a-ring>
</a-entity>

      <a-entity id="fishing-hole" gltf-model="#fishing-hole-glb" position="0 0 0" scale="0.1 0.1 0.1" visible="false"></a-entity>

      <a-sphere id="bobber" radius="0.04" color="#FF0000" visible="false"></a-sphere>
      
      <a-text id="ui-text" value="" scale="0.6 0.6 0.6" color="#FFFFFF" align="center" visible="false">
        <a-plane color="#000000" opacity="0.5" width="2" height="0.3" position="0 0 -0.01"></a-plane>
      </a-text>

      <a-entity light="type: ambient; intensity: 0.6;"></a-entity>
      <a-entity light="type: directional; intensity: 0.5;" position="2 4 1"></a-entity>

    </a-scene>
  </body>
</html>
