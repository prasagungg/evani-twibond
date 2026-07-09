<script lang="ts">
  import Navbar from './lib/Navbar.svelte';
  
  let videoElement: HTMLVideoElement;
  let canvasElement: HTMLCanvasElement;
  let resultImage: string | null = null;
  let cameraActive = false;
  let stream: MediaStream | null = null;

  async function startCamera() {
    try {
      stream = await navigator.mediaDevices.getUserMedia({ 
        video: { 
          aspectRatio: 1, 
          width: { ideal: 1000 },
          height: { ideal: 1000 }
        } 
      });
      if (videoElement) {
        videoElement.srcObject = stream;
        videoElement.play();
        cameraActive = true;
        resultImage = null; // reset if there was a previous photo
      }
    } catch (err) {
      console.error("Error accessing camera: ", err);
      alert("Gagal mengakses kamera. Pastikan Anda telah memberikan izin.");
    }
  }

  function takePhoto() {
    if (!videoElement || !canvasElement) return;

    // Set canvas size to a nice square resolution
    const size = 1000;
    canvasElement.width = size;
    canvasElement.height = size;
    
    const ctx = canvasElement.getContext('2d');
    if (!ctx) return;

    // Draw the video frame to the canvas
    // We want to crop it to a square in case the stream isn't perfectly square
    const videoAspectRatio = videoElement.videoWidth / videoElement.videoHeight;
    let sx = 0, sy = 0, sw = videoElement.videoWidth, sh = videoElement.videoHeight;
    
    if (videoAspectRatio > 1) {
      // Landscape video, crop sides
      sw = videoElement.videoHeight;
      sx = (videoElement.videoWidth - sw) / 2;
    } else if (videoAspectRatio < 1) {
      // Portrait video, crop top/bottom
      sh = videoElement.videoWidth;
      sy = (videoElement.videoHeight - sh) / 2;
    }
    
    // Draw cropped video frame (mirrored)
    ctx.save();
    ctx.scale(-1, 1);
    ctx.translate(-size, 0);
    ctx.drawImage(videoElement, sx, sy, sw, sh, 0, 0, size, size);
    ctx.restore();

    // Draw the Twibbon overlay
    const img = new Image();
    img.crossOrigin = "anonymous";
    img.onload = () => {
      ctx.drawImage(img, 0, 0, size, size);
      // Save result as base64 data URL
      resultImage = canvasElement.toDataURL('image/png');
      
      // Stop the camera since we took the photo
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
        cameraActive = false;
      }
    };
    img.src = '/twibbon.svg';
  }
  
  function retakePhoto() {
    resultImage = null;
    startCamera();
  }
</script>

<main>
  <Navbar />
  
  <div class="layout-container">
    <!-- Header Section -->
    <div class="header-section">
      <div class="header-text">
        <h1 class="title-outline">Evani</h1>
        <h2 class="title-red">Twibbon</h2>
      </div>
      <div class="divider"></div>
      <p class="description">
        Tunjukkan dukunganmu! Ambil foto langsung dari kameramu dan gunakan Twibbon resmi Evani Community.
      </p>
    </div>
    
    <!-- Preview Section -->
    <div class="right-panel">
      <div class="preview-container">
        <!-- Live Video View -->
        <div class="camera-wrapper" class:hidden={resultImage !== null}>
          <!-- svelte-ignore a11y_media_has_caption -->
          <video bind:this={videoElement} class="video-feed" playsinline></video>
          
          {#if cameraActive}
            <!-- Overlay shown during live preview -->
            <img src="/twibbon.svg" alt="Twibbon Overlay" class="overlay" />
          {:else}
            <div class="placeholder-overlay">
              <p>Kamera belum aktif</p>
            </div>
          {/if}
        </div>
        
        <!-- Result View -->
        {#if resultImage}
          <div class="result-wrapper">
            <img src={resultImage} alt="Hasil Twibbon" class="result-image" />
          </div>
        {/if}
        
        <!-- Hidden Canvas for processing -->
        <canvas bind:this={canvasElement} class="hidden"></canvas>
      </div>
      
      <!-- Decorative Ornaments -->
      <div class="ornament ornament-1"></div>
      <div class="ornament ornament-2"></div>
    </div>

    <!-- Controls Section -->
    <div class="card">
      <div class="actions">
        {#if !cameraActive && !resultImage}
          <button class="btn-primary main-btn" on:click={startCamera}>
            Nyalakan Kamera
          </button>
        {:else if cameraActive}
          <button class="btn-primary main-btn" on:click={takePhoto}>
            Ambil Foto / Jepret!
          </button>
        {:else if resultImage}
          <a href={resultImage} download="Evani-Twibbon.png" class="btn-primary main-btn download-btn">
            Download Hasil
          </a>
          <button class="btn-outline main-btn mt-3" on:click={retakePhoto}>
            Ulangi Foto
          </button>
        {/if}
      </div>
    </div>
  </div>
</main>

<style>
  main {
    padding-top: 100px; /* Space for navbar */
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .layout-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 32px;
    width: 100%;
    max-width: 480px;
    padding: 20px;
    margin: 0 auto;
    min-height: calc(100vh - 120px);
  }

  /* Header Section */
  .header-section {
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }

  .header-text {
    margin-bottom: 16px;
  }

  .title-outline {
    font-size: 3.5rem;
    color: transparent;
    -webkit-text-stroke: 2px var(--text-dark);
    line-height: 1;
    text-transform: uppercase;
    letter-spacing: 2px;
  }

  .title-red {
    font-size: 2.5rem;
    color: var(--primary-red);
    line-height: 1;
    margin-top: -8px;
    text-transform: uppercase;
  }

  .divider {
    width: 60px;
    height: 4px;
    background-color: var(--primary-red);
    margin-bottom: 16px;
    border-radius: 2px;
  }

  .description {
    font-size: 1.05rem;
    color: #666;
    line-height: 1.5;
  }

  /* Card / Controls */
  .card {
    background: white;
    border-radius: var(--radius-box);
    padding: 32px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.08);
    width: 100%;
    position: relative;
    z-index: 10;
  }

  .actions {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .main-btn {
    width: 100%;
    padding: 16px;
    font-size: 1.1rem;
    text-align: center;
  }
  
  .download-btn {
    display: block;
    text-decoration: none;
    box-sizing: border-box;
  }

  /* Right Panel / Preview Section */
  .right-panel {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    background: radial-gradient(circle at center, #fde4e5 0%, transparent 70%);
    width: 100%;
    padding: 20px 0;
  }

  .preview-container {
    position: relative;
    width: 100%;
    max-width: 400px;
    aspect-ratio: 1 / 1;
    border-radius: var(--radius-box);
    background-color: var(--secondary-pink);
    box-shadow: 0 0 40px rgba(205, 139, 142, 0.4);
    z-index: 5;
  }

  .camera-wrapper, .result-wrapper {
    width: 100%;
    height: 100%;
    position: relative;
    border-radius: var(--radius-box);
    overflow: hidden;
  }

  .video-feed {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transform: scaleX(-1); /* Mirror effect for webcam */
  }

  .overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 10;
  }

  .placeholder-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(0,0,0,0.1);
    color: white;
    font-weight: bold;
    font-size: 1.2rem;
  }

  .result-image {
    width: 100%;
    height: 100%;
    object-fit: contain;
    border-radius: var(--radius-box);
  }

  .hidden {
    display: none;
  }

  /* Ornaments */
  .ornament {
    position: absolute;
    border-radius: 50%;
    background-color: var(--primary-red);
    opacity: 0.1;
    z-index: 1;
  }

  .ornament-1 {
    width: 150px;
    height: 150px;
    top: -10px;
    right: -20px;
  }

  .ornament-2 {
    width: 200px;
    height: 200px;
    bottom: -30px;
    left: -30px;
    background-color: var(--secondary-pink);
    opacity: 0.2;
  }
</style>
