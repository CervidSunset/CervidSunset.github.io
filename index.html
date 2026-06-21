<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Liminal Poolrooms Fishing</title>
    <script src="https://aframe.io/releases/1.4.2/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/gh/c-frame/aframe-extras@7.2.0/dist/aframe-extras.min.js"></script>
    
    <script>
      // 1. INFINITE PROCEDURAL GENERATOR (Proximity Rendering)
AFRAME.registerComponent('infinite-poolrooms', {
  schema: {
    tileSize: { type: 'number', default: 4 },
    renderRadius: { type: 'number', default: 5 } // Renders a 5x5 tile radius around player
  },

  init: function () {
    this.player = document.querySelector('#rig');
    
    // Memory mapping: Stores the "seed" data for the whole world
    // Key: "x,z", Value: "tileType"
    this.worldMemory = new Map(); 
    
    // Active DOM mapping: Stores the tiles currently rendered in the scene
    this.activeTiles = new Map(); 
    
    this.lastGridPos = { x: 0, z: 0 };
    this.tickCounter = 0;
    
    this.TILE_TYPES = ['water', 'dry', 'pillar', 'wall'];
    
    // Protect the spawn tile so it never gets overwritten by procedural logic
    this.worldMemory.set('0,0', 'spawn'); 

    // Initial render
    this.updateWorldChunks();
  },

  // Tick runs continuously. We throttle it to only check chunks every ~30 frames.
  tick: function () {
    this.tickCounter++;
    if (this.tickCounter % 30 !== 0) return; 

    // Calculate player's current grid coordinate
    let pos = this.player.object3D.position;
    let gridX = Math.round(pos.x / this.data.tileSize);
    let gridZ = Math.round(pos.z / this.data.tileSize);

    // If player crossed into a new grid tile, update the world
    if (gridX !== this.lastGridPos.x || gridZ !== this.lastGridPos.z) {
      this.lastGridPos = { x: gridX, z: gridZ };
      this.updateWorldChunks();
    }
  },

  updateWorldChunks: function () {
    let px = this.lastGridPos.x;
    let pz = this.lastGridPos.z;
    let radius = this.data.renderRadius;

    // STEP 1: Generate & Render tiles around the player
    for (let x = px - radius; x <= px + radius; x++) {
      for (let z = pz - radius; z <= pz + radius; z++) {
        let key = `${x},${z}`;

        // If this coordinate has never been explored, decide what tile it is and save it to memory
        if (!this.worldMemory.has(key)) {
          let randomType = this.TILE_TYPES[Math.floor(Math.random() * this.TILE_TYPES.length)];
          this.worldMemory.set(key, randomType);
        }

        // If the tile is in memory but NOT currently rendered, render it
        // (We skip 'spawn' because it's hardcoded into the HTML)
        let tileType = this.worldMemory.get(key);
        if (!this.activeTiles.has(key) && tileType !== 'spawn') {
          this.spawnTileModel(x, z, tileType, key);
        }
      }
    }

    // STEP 2: Culling - Delete tiles that are too far away
    for (let [key, domElement] of this.activeTiles.entries()) {
      let coords = key.split(',');
      let tx = parseInt(coords[0]);
      let tz = parseInt(coords[1]);

      // If the tile is further than the render radius, remove it from the scene
      if (Math.abs(tx - px) > radius + 1 || Math.abs(tz - pz) > radius + 1) {
        domElement.parentNode.removeChild(domElement); // Remove from A-Frame scene
        this.activeTiles.delete(key);                  // Remove from active list
      }
    }
  },

  spawnTileModel: function (x, z, type, key) {
    let tileEl = document.createElement('a-entity');
    tileEl.setAttribute('position', `${x * this.data.tileSize} 0 ${z * this.data.tileSize}`);

    // PLACEHOLDERS (Replace innerHTML with your .glb files later)
    // e.g., tileEl.setAttribute('gltf-model', `#tile-${type}`);
    if (type === 'water') {
      tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#1CA3EC" material="opacity: 0.8"></a-plane>
                          <a-plane position="0 -0.5 0" rotation="-90 0 0" width="4" height="4" color="#F0F8FF"></a-plane>`;
    } else if (type === 'dry') {
      tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#F0F8FF" material="roughness: 0.2"></a-plane>`;
    } else if (type === 'pillar') {
      tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#1CA3EC" material="opacity: 0.8"></a-plane>
                          <a-box position="0 1.5 0" width="0.8" height="3" depth="0.8" color="#FFFFFF"></a-box>`;
    } else if (type === 'wall') {
      tileEl.innerHTML = `<a-plane rotation="-90 0 0" width="4" height="4" color="#F0F8FF"></a-plane>
                          <a-box position="0 2 -1.9" width="4" height="4" depth="0.2" color="#FFFFFF"></a-box>`;
    }

    this.el.appendChild(tileEl);
    this.activeTiles.set(key, tileEl);
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
