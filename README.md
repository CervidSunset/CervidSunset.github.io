<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Liminal Poolrooms Fishing</title>
    <script src="https://aframe.io/releases/1.4.2/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/gh/c-frame/aframe-extras@7.2.0/dist/aframe-extras.min.js"></script>
    
    <script>
      // 1. PROCEDURAL TILE GENERATOR (WFC Framework)
      AFRAME.registerComponent('wfc-generator', {
        init: function () {
          const gridSize = 12; // Generates a 24x24 grid
          const tileSize = 4;  // 4x4 meter tiles

          // Tile Types (Map these to your .glb files later)
          const TILE_TYPES = ['water', 'dry', 'pillar', 'wall'];

          for (let x = -gridSize; x <= gridSize; x++) {
            for (let z = -gridSize; z <= gridSize; z++) {
              // Skip the 0,0 coordinate to leave room for the Spawn Tile
              if (x === 0 && z === 0) continue;

              const posX = x * tileSize;
              const posZ = z * tileSize;
              
              // Basic adjacency logic (Simplified WFC)
              // In a full implementation, this checks neighboring tiles before selecting.
              let selectedTile = TILE_TYPES[Math.floor(Math.random() * TILE_TYPES.length)];
              
              let tileEl = document.createElement('a-entity');
              tileEl.setAttribute('position', `${posX} 0 ${posZ}`);

              // PLACEHOLDER GENERATION
              // Replace these blocks with: tileEl.setAttribute('gltf-model', '#your-model-id');
              if (selectedTile === 'water') {
                tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#1CA3EC" material="opacity: 0.8"></a-plane>
                                    <a-plane position="0 -0.5 0" rotation="-90 0 0" width="4" height="4" color="#F0F8FF"></a-plane>`;
              } else if (selectedTile === 'dry') {
                tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#F0F8FF" material="roughness: 0.2"></a-plane>`;
              } else if (selectedTile === 'pillar') {
                tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#1CA3EC" material="opacity: 0.8"></a-plane>
                                    <a-box position="0 1.5 0" width="0.8" height="3" depth="0.8" color="#FFFFFF"></a-box>`;
              } else if (selectedTile === 'wall') {
                tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#F0F8FF"></a-plane>
                                    <a-box position="0 2 -1.9" width="4" height="4" depth="0.2" color="#FFFFFF"></a-box>`;
              }

              this.el.appendChild(tileEl);
            }
          }
        }
      });

      // 2. CORE FISHING MECHANIC
      AFRAME.registerComponent('fishing-system', {
        init: function () {
          this.state = 'idle'; // idle, waiting, hooked
          this.rodEquipped = false;
          this.fishVarieties = [
            "Ethereal Koi", "Tile-Glider", "Liminal Bass", 
            "Chlorine Guppy", "Backrooms Snapper"
          ];
          this.uiText = document.querySelector('#fishing-ui');
          this.el.addEventListener('triggerdown', this.onTrigger.bind(this));
        },
        
        onTrigger: function (evt) {
          if (!this.rodEquipped) {
            const intersections = this.el.components.raycaster.intersectedEls;
            if (intersections.length > 0 && intersections[0].id === 'fishing-rod') {
              this.equipRod(intersections[0]);
            }
            return;
          }

          if (this.state === 'idle') {
            this.castLine();
          } else if (this.state === 'hooked') {
            this.catchFish();
          }
        },

        equipRod: function(rodEl) {
          this.rodEquipped = true;
          this.uiText.setAttribute('value', 'Rod equipped. Press Trigger near water to cast.');
          
          // Attach rod to controller
          rodEl.removeAttribute('position');
          rodEl.removeAttribute('rotation');
          this.el.appendChild(rodEl);
          rodEl.setAttribute('position', '0 -0.1 -0.4');
          rodEl.setAttribute('rotation', '45 0 0');
        },

        castLine: function() {
          this.state = 'waiting';
          this.uiText.setAttribute('value', 'Waiting for a bite...');
          
          // Random bite time (3 to 8 seconds)
          const biteTime = Math.floor(Math.random() * 5000) + 3000;
          
          setTimeout(() => {
            if (this.state === 'waiting') {
              this.state = 'hooked';
              // NO TIMEOUT HERE: The fish remains hooked indefinitely until player triggers.
              this.uiText.setAttribute('value', 'HOOKED! Press trigger to reel in.');
              this.uiText.setAttribute('color', '#FF4500'); 
            }
          }, biteTime);
        },

        catchFish: function() {
          const caughtFish = this.fishVarieties[Math.floor(Math.random() * this.fishVarieties.length)];
          this.state = 'idle';
          this.uiText.setAttribute('value', `Caught: ${caughtFish}!\nPress trigger to cast again.`);
          this.uiText.setAttribute('color', '#00FF00');

          this.spawnFishPlaceholder(caughtFish);
        },

        spawnFishPlaceholder: function(name) {
          let fish = document.createElement('a-cylinder');
          fish.setAttribute('radius', '0.05');
          fish.setAttribute('height', '0.2');
          fish.setAttribute('color', '#FFD700');
          fish.setAttribute('rotation', '0 0 90');
          // Spawn above desk
          fish.setAttribute('position', '0 1 -1');
          
          setTimeout(() => {
            if(fish.parentNode) fish.parentNode.removeChild(fish);
            this.uiText.setAttribute('color', '#333');
            this.uiText.setAttribute('value', 'Ready to cast.');
          }, 4000);
          
          document.querySelector('a-scene').appendChild(fish);
        }
      });
    </script>
  </head>
  <body>
    <a-assets>
      </a-assets>

    <a-scene background="color: #E0FFFF" fog="type: exponential; color: #E0FFFF; density: 0.04">
      
      <a-entity wfc-generator></a-entity>

      <a-light type="ambient" color="#FFF" intensity="0.7"></a-light>
      <a-light type="directional" position="-1 2 1" intensity="0.3"></a-light>

      <a-entity id="rig" movement-controls="speed: 0.15" position="0 0 0">
        <a-camera position="0 1.6 0">
          <a-text id="fishing-ui" value="Grab the rod with your trigger." position="0 -0.3 -1" align="center" width="1.2" color="#333"></a-text>
        </a-camera>
        
        <a-entity laser-controls="hand: left" raycaster="objects: .interactable; far: 4"></a-entity>
        <a-entity laser-controls="hand: right" raycaster="objects: .interactable; far: 4" fishing-system></a-entity>
      </a-entity>

      <a-entity id="spawn-tile" position="0 0 0">
        <a-plane rotation="-90 0 0" width="4" height="4" color="#F0F8FF"></a-plane>
        
        <a-box position="0 0.4 -1" width="1.2" height="0.8" depth="0.6" color="#8B4513"></a-box>

        <a-plane position="1.2 -0.1 0" rotation="-90 0 0" width="1.5" height="2" color="#1CA3EC" material="opacity: 0.8"></a-plane>
        <a-box position="1.2 -0.6 0" width="1.5" height="1" depth="2" color="#E0FFFF" material="side: back"></a-box>

        <a-cylinder id="fishing-rod" class="interactable" position="0.6 0.8 -0.5" radius="0.015" height="1.2" rotation="45 0 0" color="#555"></a-cylinder>
      </a-entity>

    </a-scene>
  </body>
</html>
