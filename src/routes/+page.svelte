<script>
    import { onMount, onDestroy } from 'svelte';
    import * as THREE from 'three';
    import * as SC from 'svelte-cubed'; 
   
    let spin = 0;
    const spinSpeed = 0.002;

    const videoConfigs = [
      {name: "spring", src: "/cube-video-spring.mp4"},
      {name: "rust", src: "/cube-video-rust.mp4"},
      {name: "bash", src: "/cube-video-bash.mp4"},
      {name: "svelte", src: "/cube-video-svelte.mp4"}
    ];

    let videoTextures = {};
    let materials = [];
    let backgroundTexture = null;

    /*
    * Attributs for 3d object
    */
    const boxGeometry = new THREE.BoxGeometry(1.5, 1.5, 1.5);
    const cameraPosition = [1, 1, 5];
    const directonalLightPosition = [-2, 3, 2];
    const ambientLightIntesity = 0.6;
    const directionalLightIntensity = 0.6;

    /*
    * Function to setup video element
    *
    * @param src: contains path to mp4 file
    * @return video: returns video element
    */
    const createVideoElement = (src) => {
      const video = document.createElement("video");
      video.src = src;
      video.loop = true;
      video.muted = true;
      video.autoplay = true;
      video.playsInline = true;
      video.crossOrigin = "anonymous";
      video.play().catch((e) => console.error(`Error playing video - ${src}: `, e));
      return video;
    }

    /*
    * Function to create ThreeJS video texture
    *
    * @param video: contains video element
    * @return texture: returns video texture
    */
    const createVideoTexture = (video) => {
      const texture = new THREE.VideoTexture(video);
      texture.minFilter = THREE.LinearFilter;
      texture.magFilter = THREE.LinearFilter;
      texture.format = THREE.RGBFormat;
      return texture;
    }

    /*
    * Creates video elements and video textures
    */
    const loadVideos = async () => {
      const videoElements = videoConfigs.map(config => ({
        name: config.name,
        video: createVideoElement(config.src)
      }));

    await Promise.all(videoElements.map(({ video }) => {
        return new Promise((resolve, reject) => {
          video.addEventListener("canplay", resolve);
          video.addEventListener("error", reject);
        });
      })).catch(e => console.error("Error loading videos: ", e ));

      videoElements.forEach(({ name, video }) => {
        videoTextures[name] = createVideoTexture(video);
      });
    };

    /*
    * Loads scene background from file
    *
    * @throws error: throws error if image fails to load
    */
    const loadBackground = () => {
      return new Promise((resolve, reject) => {
        const loader = new THREE.TextureLoader();
        loader.load(
          "/background-image.jpg",
          (texture) => {
            texture.minFilter = THREE.LinearFilter;
            texture.magFilter = THREE.LinearFilter;
            texture.format = THREE.RGBFormat;
            backgroundTexture = texture;
            resolve();
          },
          undefined,
          (e) => {
            console.error("Error occured when loading background image: ", e);
            reject(e);
          }
        );
      });
    };

    /*
    * Maps videos to sides of cube
    */
    const setupMaterials = () => {
      materials = [
        new THREE.MeshStandardMaterial({ map: videoTextures.spring }), 
        new THREE.MeshStandardMaterial({ map: videoTextures.rust }),
        new THREE.MeshStandardMaterial({ map: videoTextures.bash }),
        new THREE.MeshStandardMaterial({ map: videoTextures.spring }),
        new THREE.MeshStandardMaterial({ map: videoTextures.svelte }),
        new THREE.MeshStandardMaterial({ map: videoTextures.bash }),
      ];
    };

    /*
    * Runs setup functions on page mount
    *
    * @throws error: throws error if setup functions fails
    */
    onMount(async () => {
      try {
        await loadVideos();
        setupMaterials();
        await loadBackground();
      } catch (e) {
        console.error("Error during initialization: ", e);
      }
    });

    /*
    * Function terminates video elements and ThreeJS cube when leaving page
    */
    onDestroy(() => {
      boxGeometry.dispose();
      materials.forEach(material => material.dispose());
      Object.values(videoTextures).forEach(texture => texture.dispose());
      if(backgroundTexture) backgroundTexture.dispose();

      videoConfigs.forEach(config => {
        const video = videoTextures[config.name]?.image;
        if(video) {
          video.pause();
          video.src = "";
          video.load();
        }
      });
    });

    /*
    * Function makes cube spin
    */
    SC.onFrame(() => {
      spin += spinSpeed;
    });
    

</script>

{#if materials.length === 6 && backgroundTexture }
    <SC.Canvas antialias background={backgroundTexture}>
        <SC.Mesh
            geometry={boxGeometry} 
            material={materials} 
            rotation={[0, spin, 0]}
        />
        <SC.PerspectiveCamera position={cameraPosition} />
        <SC.OrbitControls enableZoom={false} />
        <SC.AmbientLight intensity={ambientLightIntesity} />
        <SC.DirectionalLight
            intensity={directionalLightIntensity}
            position={directonalLightPosition}
        />
    </SC.Canvas>
{/if}
