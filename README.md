<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Desk Fishing Hole</title>
    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
    
    <script>
      AFRAME.registerComponent('game-manager', {
        init: function () {
          this.state = 'placing'; // States: placing, adjusting, idle, casted, bite, caught
          
          this.bobber = document.querySelector('#bobber');
          this.fishText = document.querySelector('#ui-text');
          this.hole = document.querySelector('#fishing-hole');
          this.reticle = document.querySelector('#reticle');
          this.startMenu = document.querySelector('#start-menu');
          this.heightMenu = document.querySelector('#height-menu');
          
          this.targetPos = {x: 0, y: 0, z: -1.5}; 

          // 1. Hover listener: Move the green ring as the user moves their controller laser
          document.querySelector('#floor-plane').addEventListener('mousemove', (evt) => {
            if (this.state === 'placing') {
              let hitPoint = evt.detail.intersection.point;
              this.targetPos.x = hitPoint.x;
              this.targetPos.z = hitPoint.z;
              
              this.reticle.setAttribute('position', `${this.targetPos.x} 0.01 ${this.targetPos.z}`);
              this.reticle.setAttribute('visible', 'true');
            }
          });

          // 2. Click listener: Lock the base X/Z coordinates when they click the floor
          document.querySelector('#floor-plane').addEventListener('click', () => {
            if (this.state === 'placing') {
              this.lockBasePosition();
            }
          });

          // 3. Height Menu listeners: Wire up the virtual adjustment buttons
          document.querySelector('#btn-raise').addEventListener('click', () => this.adjustHeight(0.02));
          document.querySelector('#btn-lower').addEventListener('click', () => this.adjustHeight(-0.02));
          document.querySelector('#btn-confirm').addEventListener('click', () => this.confirmPlacement());

          // 4. Global Action trigger: Handles actual fishing gameplay mechanics
          const rightHand = document.querySelector('#rightHand');
          rightHand.addEventListener('triggerdown', (evt) => {
            // Only capture raw trigger pulls if we are actively fishing
            if (this.state !== 'placing' && this.state !== 'adjusting') {
              this.handleFishingGameplay();
            }
          });
        },

        lockBasePosition: function() {
          this.state = 'adjusting';
          
          // Spawn pond at baseline height (0) and make it visible
          this.hole.setAttribute('position', this.targetPos);
          this.hole.setAttribute('visible', 'true');
          
          // Hide start instructions, show the height calibration panel
          this.startMenu.setAttribute('visible', 'false');
          this.heightMenu.setAttribute('visible', 'true');
          
          // Turn off the floor plane so it doesn't block our menu clicks
          document.querySelector('#floor-plane').setAttribute('class', '');
        },

        adjustHeight: function(amount) {
          if (this.state === 'adjusting') {
            this.targetPos.y += amount;
            
            // Move both the pond and the alignment ring up/down in real-time
            this.hole.setAttribute('position', this.targetPos);
            this.reticle.setAttribute('position', this.targetPos);
          }
        },

        confirmPlacement: function() {
          this.state = 'idle';
          
          // Hide setup UIs entirely
          this.heightMenu.setAttribute('visible', 'false');
          this.reticle.setAttribute('visible', 'false');
          
          // Place fishing UI text floating comfortably above the customized pond height
          this.fishText.setAttribute('position', {
             x: this.targetPos.x,
             y: this.targetPos.y + 0.8,
             z: this.targetPos.z
          });
          this.fishText.setAttribute('value', 'Position Locked!\nPull right trigger to cast line.');
          this.fishText.setAttribute('visible', 'true');
        },

        handleFishingGameplay: function () {
          if (this.state === 'idle') this.castLine();
          else if (this.state === 'casted') this.reelEarly();
          else if (this.state === 'bite') this.catchFish();
          else if (this.state === 'caught') this.resetToIdle();
        },

        castLine: function () {
          this.state = 'casted';
          this.fishText.setAttribute('value', 'Line casted... waiting for a bite.');
          
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
          
          this.fishText.setAttribute('value', `Caught: ${randomFish}!\nPull trigger to recast.`);
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
          this.fishText.setAttribute('value', 'The fish got away! Try again.');
          this.bobber.setAttribute('visible', 'false');
        },

        resetToIdle: function () {
          this.state = 'idle';
          this.fishText.setAttribute('value', 'Pull trigger to cast into the hole.');
        }
      });
    </script>
  </head>
  <body>

    <a-scene 
      webxr="optionalFeatures: local-floor; referenceSpaceType: local-floor"
      xr-mode-ui="XRMode: xr"
      background="transparent: true"
      game-manager>
      
      <a-assets>
        <a-asset-item id="fishing-hole-glb" src="./models/fishing_hole.glb"></a-asset-item>
      </a-assets>

      <a-entity id="start-menu" position="0 1.4 -1.2">
        <a-plane color="#111111" opacity="0.9" width="1.6" height="0.6" position="0 0 -0.01"></a-plane>
        <a-text value="Welcome to Desk Fishing!\n\nAim your laser at the ground or desk\nand click to set the location." 
                align="center" color="#FFFFFF" width="1.4"></a-text>
      </a-entity>

      <a-entity id="height-menu" position="0 1.4 -1.2" visible="false">
        <a-plane color="#111111" opacity="0.9" width="1.6" height="0.7" position="0 0 -0.01"></a-plane>
        <a-text value="Calibrate Pond Height\nUse your laser pointer to click the buttons." align="center" color="#FFFFFF" width="1.4" position="0 0.22 0"></a-text>
        
        <a-box id="btn-raise" class="clickable" color="#4CAF50" position="-0.4 -0.1 0" width="0.3" height="0.15" depth="0.02">
          <a-text value="HIGHER" align="center" position="0 0 0.015" scale="0.4 0.4 0.4" color="#FFF"></a-text>
        </a-box>
        <a-box id="btn-lower" class="clickable" color="#F44336" position="0 -0.1 0" width="0.3" height="0.15" depth="0.02">
          <a-text value="LOWER" align="center" position="0 0 0.015" scale="0.4 0.4 0.4" color="#FFF"></a-text>
        </a-box>
        <a-box id="btn-confirm" class="clickable" color="#008CBA" position="0.4 -0.1 0" width="0.3" height="0.15" depth="0.02">
          <a-text value="CONFIRM" align="center" position="0 0 0.015" scale="0.4 0.4 0.4" color="#FFF"></a-text>
        </a-box>
      </a-entity>

      <a-entity id="leftHand" laser-controls="hand: left" raycaster="objects: .clickable, .interact-floor; far: 5"></a-entity>
      <a-entity id="rightHand" laser-controls="hand: right" raycaster="objects: .clickable, .interact-floor; far: 5"></a-entity>

      <a-plane id="floor-plane" class="interact-floor" position="0 0 0" rotation="-90 0 0" width="30" height="30" material="opacity: 0; transparent: true"></a-plane>

      <a-ring id="reticle" rotation="-90 0 0" radius-inner="0.12" radius-outer="0.15" color="#00FF00" visible="false"></a-ring>
      
      <a-entity id="fishing-hole" gltf-model="#fishing-hole
