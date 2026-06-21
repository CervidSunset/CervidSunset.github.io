<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Liminal Poolrooms Fishing - Models Integration</title>
    <script src="https://aframe.io/releases/1.4.2/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/gh/c-frame/aframe-extras@7.2.0/dist/aframe-extras.min.js"></script>
    
    <script>
      // 1. INFINITE CHUNK MANAGER (Now with GLTF Support)
      AFRAME.registerComponent('infinite-chunk-manager', {
        schema: {
          tileSize: {type: 'number', default: 4},
          renderRadius: {type: 'int', default: 5},
          cullRadius: {type: 'int', default: 7}
        },

        init: function () {
          this.camera = document.querySelector('[camera]');
          this.activeTiles = {}; 
          this.tickThrottle = 0;
          this.TILE_TYPES = ['water', 'dry', 'pillar', 'wall'];
        },

        tick: function (time, timeDelta) {
          this.tickThrottle += timeDelta;
          if (this.tickThrottle > 500) {
            this.updateGrid();
            this.tickThrottle = 0;
          }
        },

        updateGrid: function () {
          const pos = this.camera.getAttribute('position');
          const gridX = Math.round(pos.x / this.data.tileSize);
          const gridZ = Math.round(pos.z / this.data.tileSize);

          // SPAWN
          for (let x = -this.data.renderRadius; x <= this.data.renderRadius; x++) {
            for (let z = -this.data.renderRadius; z <= this.data.renderRadius; z++) {
              const curX = gridX + x;
              const curZ = gridZ + z;
              const key = `${curX},${curZ}`;
              
              if (curX === 0 && curZ === 0) continue; 
              
              if (!this.activeTiles[key]) {
                this.spawnTile(curX, curZ, key);
              }
            }
          }

          // CULL
          for (let key in this.activeTiles) {
            let coords = key.split(',');
            let tileX = parseInt(coords[0]);
            let tileZ = parseInt(coords[1]);
            
            if (Math.abs(tileX - gridX) > this.data.cullRadius || Math.abs(tileZ - gridZ) > this.data.cullRadius) {
              if (tileX === 0 && tileZ === 0) continue; 
              
              this.el.removeChild(this.activeTiles[key]);
              delete this.activeTiles[key];
            }
          }
        },

        spawnTile: function(gridX, gridZ, key) {
          const posX = gridX * this.data.tileSize;
          const posZ = gridZ * this.data.tileSize;
          let selectedTile = this.TILE_TYPES[Math.floor(Math.random() * this.TILE_TYPES.length)];
          
          let tileEl = document.createElement('a-entity');
          tileEl.setAttribute('position', `${posX} 0 ${posZ}`);

          // Global primitive ceiling for the claustrophobic feel
          let ceilingHTML = `<a-plane position="0 4 0" rotation="90 0 0" width="4" height="4" color="#FFFFF0" material="emissive: #fff; emissiveIntensity: 0.15"></a-plane>`;

          // Inject the correct GLTF model based on the procedural selection
          if (selectedTile === 'water') {
            // Note: Keeping the water plane as requested, hovering slightly above the model's floor
            tileEl.innerHTML = ceilingHTML + 
              `<a-gltf-model src="#tile-water" position="0 0 0"></a-gltf-model>
               <a-plane position="0 0.1 0" rotation="-90 0 0" width="4" height="4" color="#1CA3EC" material="opacity: 0.8; transparent: true"></a-plane>`;
          } else if (selectedTile === 'dry') {
            tileEl.innerHTML = ceilingHTML + 
              `<a-gltf-model src="#tile-dry" position="0 0 0"></a-gltf-model>`;
          } else if (selectedTile === 'pillar') {
            // Assuming the pillar sits in water for the aesthetic, adding the water plane here too
            tileEl.innerHTML = ceilingHTML + 
              `<a-gltf-model src="#tile-pillar" position="0 0 0"></a-gltf-model>
               <a-plane position="0 0.1 0" rotation="-90 0 0" width="4" height="4" color="#1CA3EC" material="opacity: 0.8; transparent: true"></a-plane>`;
          } else if (selectedTile === 'wall') {
            tileEl.innerHTML = ceilingHTML + 
              `<a-gltf-model src="#tile-wall" position="0 0 0"></a-gltf-model>`;
          }

          this.el.appendChild(tileEl);
          this.activeTiles[key] = tileEl; 
        }
      });

      // 2. CORE FISHING MECHANIC (Unchanged)
      AFRAME.registerComponent('fishing-system', {
        init: function () {
          this.state = 'idle'; 
          this.rodEquipped = false;
          this.fishVarieties = ["Ethereal Koi", "Tile-Glider", "Liminal Bass", "Chlorine Guppy", "Backrooms Snapper"];
          this.uiText = document.querySelector('#fishing-ui');
          this.el.addEventListener('triggerdown', this.onTrigger.bind(this));
        },
        
        onTrigger: function (evt) {
          if (!this.rodEquipped) {
            const intersections = this.el.components.raycaster.intersectedEls;
            if (intersections.length > 0 && intersections[0].id === 'fishing-rod-model') {
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
          rodEl.removeAttribute('position');
          rodEl.removeAttribute('rotation');
          this.el.appendChild(rodEl);
          rodEl.setAttribute('position', '0 -0.1 -0.4');
          rodEl.setAttribute('rotation', '45 0 0');
        },

        castLine: function() {
          this.state = 'waiting';
          this.uiText.setAttribute('value', 'Waiting for a bite...');
          const biteTime = Math.floor(Math.random() * 5000) + 3000;
          
          setTimeout(() => {
            if (this.state === 'waiting') {
              this.state = 'hooked';
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
          // You can replace this cylinder with <a-gltf-model> once you make the fish in Blender
          let fish = document.createElement('a-cylinder');
          fish.setAttribute('radius', '0.05');
          fish.setAttribute('height', '0.2');
          fish.setAttribute('color', '#FFD700');
          fish.setAttribute('rotation', '0 0 90');
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
      <a-asset-item id="tile-dry" src="tile-dry.glb"></a-asset-item>
      <a-asset-item id="tile-water" src="tile-water.glb"></a-asset-item>
      <a-asset-item id="tile-pillar" src="tile-pillar.glb"></a-asset-item>
      <a-asset-item id="tile-wall" src="tile-wall.glb"></a-asset-item>
      
      <a-asset-item id="desk" src="desk.glb"></a-asset-item>
      <a-asset-item id="rod" src="fishing-rod.glb"></a-asset-item>
    </a-assets>

    <a-scene background="color: #E0FFFF" fog="type: exponential; color: #E0FFFF; density: 0.04">
      
      <a-entity infinite-chunk-manager></a-entity>

      <a-light type="ambient" color="#FFF" intensity="0.7"></a-light>
      <a-entity light="type: directional; intensity: 0.3; position: -1 2 1" position="0 0 0" id="follow-light"></a-entity>

      <a-entity id="rig" movement-controls="speed: 0.15" position="0 0 0">
        <a-camera position="0 1.6 0">
          <a-text id="fishing-ui" value="Grab the rod with your trigger." position="0 -0.3 -1" align="center" width="1.2" color="#333"></a-text>
        </a-camera>
        <a-entity laser-controls="hand: left" raycaster="objects: .interactable; far: 4"></a-entity>
        <a-entity laser-controls="hand: right" raycaster="objects: .interactable; far: 4" fishing-system></a-entity>
      </a-entity>

      <a-entity id="spawn-tile" position="0 0 0">
        <a-plane position="0 4 0" rotation="90 0 0" width="4" height="4" color="#FFFFF0" material="emissive: #fff; emissiveIntensity: 0.15"></a-plane>
        
        <a-plane rotation="-90 0 0" width="4" height="4" color="#F0F8FF"></a-plane>
        
        <a-gltf-model src="#desk" position="0 0.4 -1"></a-gltf-model>

        <a-plane position="1.2 -0.1 0" rotation="-90 0 0" width="1.5" height="2" color="#1CA3EC" material="opacity: 0.8; transparent: true"></a-plane>
        <a-box position="1.2 -0.6 0" width="1.5" height="1" depth="2" color="#E0FFFF" material="side: back"></a-box>

        <a-gltf-model id="fishing-rod-model" src="#rod" class="interactable" position="0.6 0.8 -0.5"></a-gltf-model>
      </a-entity>

      <script>
        document.querySelector('#rig').addEventListener('componentchanged', function (evt) {
          if (evt.detail.name === 'position') {
            document.querySelector('#follow-light').setAttribute('position', evt.detail.newData);
          }
        });
      </script>

    </a-scene>
  </body>
</html>
