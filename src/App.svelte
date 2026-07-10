<script lang="ts">
  import Navbar from './lib/Navbar.svelte';
  
  let videoElement: HTMLVideoElement;
  let canvasElement: HTMLCanvasElement;
  let resultImage: string | null = null;
  let cameraActive = false;
  let stream: MediaStream | null = null;
  const twibbons = ['/TWIBBONEVANI.png', '/TWIBBONEVANI2.png'];
  let selectedTwibbon = twibbons[0];
  let fileInput: HTMLInputElement;
  
  // Editor State
  let editMode = false;
  let currentImageSrc: string | null = null;
  let scale = 1.0;
  let offsetX = 0;
  let offsetY = 0;
  let isDragging = false;
  let startX = 0;
  let startY = 0;
  let previewWidth = 400; // default, will bind to clientWidth

  async function startCamera() {
    try {
      stream = await navigator.mediaDevices.getUserMedia({ 
        video: { 
          aspectRatio: 1, 
          width: { ideal: 1000 },
          height: { ideal: 1000 },
          facingMode: 'user'
        } 
      });
      if (videoElement) {
        videoElement.srcObject = stream;
        videoElement.play();
        cameraActive = true;
        resultImage = null;
        editMode = false;
      }
    } catch (err) {
      console.error("Error accessing camera: ", err);
      alert("Gagal mengakses kamera. Pastikan Anda telah memberikan izin.");
    }
  }

  function takePhoto() {
    if (!videoElement || !canvasElement) return;

    canvasElement.width = videoElement.videoWidth;
    canvasElement.height = videoElement.videoHeight;
    const ctx = canvasElement.getContext('2d');
    if (!ctx) return;
    
    // Mirror the capture so the user sees themselves as in a mirror
    ctx.save();
    ctx.scale(-1, 1);
    ctx.translate(-canvasElement.width, 0);
    ctx.drawImage(videoElement, 0, 0);
    ctx.restore();
    
    currentImageSrc = canvasElement.toDataURL('image/png');
    editMode = true;
    scale = 1.0;
    offsetX = 0;
    offsetY = 0;
    
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
      cameraActive = false;
    }
  }

  function handleFileUpload(event: Event) {
    const target = event.target as HTMLInputElement;
    if (!target.files || target.files.length === 0) return;
    
    const file = target.files[0];
    const reader = new FileReader();
    reader.onload = (e) => {
      if (!e.target || typeof e.target.result !== 'string') return;
      currentImageSrc = e.target.result;
      editMode = true;
      scale = 1.0;
      offsetX = 0;
      offsetY = 0;
    };
    reader.readAsDataURL(file);
    
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
      cameraActive = false;
    }
    // reset input
    target.value = '';
  }

  function onPointerDown(e: MouseEvent | TouchEvent) {
    if (!editMode) return;
    isDragging = true;
    const clientX = 'touches' in e ? e.touches[0].clientX : (e as MouseEvent).clientX;
    const clientY = 'touches' in e ? e.touches[0].clientY : (e as MouseEvent).clientY;
    startX = clientX - offsetX;
    startY = clientY - offsetY;
  }
  
  function onPointerMove(e: MouseEvent | TouchEvent) {
    if (!isDragging || !editMode) return;
    const clientX = 'touches' in e ? e.touches[0].clientX : (e as MouseEvent).clientX;
    const clientY = 'touches' in e ? e.touches[0].clientY : (e as MouseEvent).clientY;
    offsetX = clientX - startX;
    offsetY = clientY - startY;
  }
  
  function onPointerUp() {
    isDragging = false;
  }

  function generateFinalImage() {
    if (!currentImageSrc || !canvasElement) return;
    
    const size = 1000;
    canvasElement.width = size;
    canvasElement.height = size;
    const ctx = canvasElement.getContext('2d');
    if (!ctx) return;
    
    const img = new Image();
    img.onload = () => {
      // Calculate drawing dimensions to match the CSS object-fit: cover logic
      const containerW = previewWidth || 400;
      
      const canvasBaseScale = Math.max(size / img.width, size / img.height);
      const drawWidth = img.width * canvasBaseScale * scale;
      const drawHeight = img.height * canvasBaseScale * scale;
      
      const cx = size / 2;
      const cy = size / 2;
      const multiplier = size / containerW;
      
      const finalX = cx - (drawWidth / 2) + (offsetX * multiplier);
      const finalY = cy - (drawHeight / 2) + (offsetY * multiplier);
      
      // Draw user image
      ctx.drawImage(img, finalX, finalY, drawWidth, drawHeight);

      // Draw overlay
      const overlayImg = new Image();
      overlayImg.crossOrigin = "anonymous";
      overlayImg.onload = () => {
        ctx.drawImage(overlayImg, 0, 0, size, size);
        resultImage = canvasElement.toDataURL('image/png');
        editMode = false;
      };
      overlayImg.src = selectedTwibbon;
    };
    img.src = currentImageSrc;
  }

  function retakePhoto() {
    resultImage = null;
    editMode = false;
    currentImageSrc = null;
    startCamera();
  }
</script>

<main>
  <Navbar />
  
  <div class="layout-container">
    <div class="header-section">
      <div class="header-text">
        <h1 class="title-outline">Evani</h1>
        <h2 class="title-red">Twibbon</h2>
      </div>
      <div class="divider"></div>
      <p class="description">
        Tunjukkan dukunganmu! Ambil foto dari kamera atau unggah dari galeri.
      </p>
    </div>
    
    <div class="right-panel">
      <!-- Bind clientWidth to previewWidth so we can map transforms accurately -->
      <div class="preview-container" bind:clientWidth={previewWidth}>
        <div class="camera-wrapper" class:hidden={resultImage !== null}>
          
          <!-- svelte-ignore a11y_media_has_caption -->
          <video 
            bind:this={videoElement} 
            class="video-feed" 
            class:hidden={!cameraActive || editMode} 
            playsinline
          ></video>

          {#if editMode && currentImageSrc}
            <!-- svelte-ignore a11y_no_static_element_interactions -->
            <div 
              class="edit-layer"
              on:mousedown={onPointerDown}
              on:mousemove={onPointerMove}
              on:mouseup={onPointerUp}
              on:mouseleave={onPointerUp}
              on:touchstart|preventDefault={onPointerDown}
              on:touchmove|preventDefault={onPointerMove}
              on:touchend|preventDefault={onPointerUp}
            >
              <img 
                src={currentImageSrc} 
                alt="Edit" 
                class="edit-image"
                style="transform: translate({offsetX}px, {offsetY}px) scale({scale});"
              />
            </div>
            <!-- Overlay remains on top, pointer-events-none so touches pass through -->
            <img src={selectedTwibbon} alt="Twibbon Overlay" class="overlay" />
          
          {:else if cameraActive}
            <img src={selectedTwibbon} alt="Twibbon Overlay" class="overlay" />
          
          {:else}
            <div class="placeholder-overlay">
              <p>Mulai Kamera atau Pilih Galeri</p>
            </div>
          {/if}

        </div>
        
        {#if resultImage}
          <div class="result-wrapper">
            <img src={resultImage} alt="Hasil Twibbon" class="result-image" />
          </div>
        {/if}
        
        <canvas bind:this={canvasElement} class="hidden"></canvas>
      </div>
      
      <div class="ornament ornament-1"></div>
      <div class="ornament ornament-2"></div>
    </div>

    <div class="card">
      <div class="twibbon-selector" class:hidden={editMode || resultImage}>
        <p class="selector-title">Pilih Bingkai:</p>
        <div class="selector-options">
          {#each twibbons as t}
            <button 
              class="twibbon-option {selectedTwibbon === t ? 'active' : ''}" 
              on:click={() => selectedTwibbon = t}
            >
              <img src={t} alt="Twibbon Option" />
            </button>
          {/each}
        </div>
      </div>
      
      {#if editMode}
        <!-- Edit Controls -->
        <div class="edit-controls">
          <p class="text-sm mb-2" style="color: #666; font-size: 0.9rem; text-align: center; margin-bottom: 8px;">
            Geser gambar untuk menyesuaikan posisi.<br/>Gunakan slider di bawah untuk memperbesar.
          </p>
          <div class="slider-container">
            <label for="zoomSlider" class="zoom-icon">🔍</label>
            <input 
              id="zoomSlider" 
              type="range" 
              min="1" 
              max="4" 
              step="0.05" 
              bind:value={scale} 
              class="zoom-slider"
            />
          </div>
          <div class="actions" style="margin-top: 16px;">
            <button class="btn-primary main-btn" style="background-color: #25D366; box-shadow: 0 4px 14px rgba(37, 211, 102, 0.3);" on:click={generateFinalImage}>
              Selesai & Terapkan
            </button>
            <button class="btn-outline main-btn" style="margin-top: 8px;" on:click={retakePhoto}>
              Batal
            </button>
          </div>
        </div>
      {:else}
        <div class="actions">
          {#if !cameraActive && !resultImage}
            <button class="btn-primary main-btn" on:click={startCamera}>
              Nyalakan Kamera
            </button>
            <button class="btn-outline main-btn" style="margin-top: 8px;" on:click={() => fileInput.click()}>
              Pilih dari Galeri
            </button>
          {:else if cameraActive}
            <button class="btn-primary main-btn" on:click={takePhoto}>
              Ambil Foto / Jepret!
            </button>
            <button class="btn-outline main-btn" style="margin-top: 8px;" on:click={() => fileInput.click()}>
              Atau Pilih Galeri
            </button>
          {:else if resultImage}
            <a href={resultImage} download="Evani-Twibbon.png" class="btn-primary main-btn download-btn">
              Download Hasil
            </a>
            <button class="btn-outline main-btn" style="margin-top: 8px;" on:click={retakePhoto}>
              Ulangi Foto
            </button>
          {/if}
        </div>
      {/if}
    </div>
  </div>
  
  <input type="file" accept="image/*" class="hidden" bind:this={fileInput} on:change={handleFileUpload} />
</main>

<style>
  main {
    padding-top: 100px;
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

  .header-section {
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }

  .header-text { margin-bottom: 16px; }

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
    width: 60px; height: 4px; background-color: var(--primary-red);
    margin-bottom: 16px; border-radius: 2px;
  }

  .description { font-size: 1.05rem; color: #666; line-height: 1.5; }

  .card {
    background: rgba(255, 255, 255, 0.5);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border: 1px solid rgba(255, 255, 255, 0.6);
    border-radius: var(--radius-box);
    padding: 32px;
    box-shadow: 0 8px 32px rgba(186, 28, 34, 0.08);
    width: 100%;
    position: relative;
    z-index: 10;
  }

  .twibbon-selector { margin-bottom: 24px; text-align: center; }
  .selector-title { font-size: 0.9rem; font-weight: 600; margin-bottom: 12px; color: #666; }
  .selector-options { display: flex; justify-content: center; gap: 16px; }

  .twibbon-option {
    width: 60px; height: 60px; border-radius: 12px; overflow: hidden;
    border: 3px solid transparent; padding: 0; background: rgba(255, 255, 255, 0.5);
    transition: transform 0.2s, border-color 0.2s;
  }
  .twibbon-option img { width: 100%; height: 100%; object-fit: cover; }
  .twibbon-option.active { border-color: var(--primary-red); transform: scale(1.1); }
  .twibbon-option:hover { transform: scale(1.05); }

  .actions { display: flex; flex-direction: column; gap: 12px; }
  .main-btn { width: 100%; padding: 16px; font-size: 1.1rem; text-align: center; }
  .download-btn { display: block; text-decoration: none; box-sizing: border-box; }

  .right-panel {
    position: relative; display: flex; align-items: center; justify-content: center;
    width: 100%; padding: 20px 0;
  }

  .preview-container {
    position: relative; width: 100%; max-width: 400px; aspect-ratio: 1 / 1;
    border-radius: var(--radius-box); background-color: var(--secondary-pink);
    box-shadow: 0 0 40px rgba(205, 139, 142, 0.4); z-index: 5;
    overflow: hidden; /* Important for pan boundary */
  }

  .camera-wrapper, .result-wrapper {
    width: 100%; height: 100%; position: relative;
    border-radius: var(--radius-box); overflow: hidden;
  }

  .video-feed {
    width: 100%; height: 100%; object-fit: cover; transform: scaleX(-1);
  }

  .overlay {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    pointer-events: none; z-index: 10;
  }

  .placeholder-overlay {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    background: rgba(0,0,0,0.1); color: white; font-weight: bold; font-size: 1.2rem;
  }

  .result-image {
    width: 100%; height: 100%; object-fit: contain; border-radius: var(--radius-box);
  }

  .hidden { display: none !important; }

  .ornament {
    position: absolute; border-radius: 50%; background-color: var(--primary-red);
    opacity: 0.1; z-index: 1;
  }
  .ornament-1 { width: 150px; height: 150px; top: -10px; right: -20px; }
  .ornament-2 {
    width: 200px; height: 200px; bottom: -30px; left: -30px;
    background-color: var(--secondary-pink); opacity: 0.2;
  }

  /* Edit Layer specific */
  .edit-layer {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    cursor: grab;
    z-index: 5; /* below overlay */
    background: #e9e9e9; /* dark background while editing looks nice */
  }
  .edit-layer:active { cursor: grabbing; }

  .edit-image {
    width: 100%; height: 100%; object-fit: cover;
    pointer-events: none; /* Let the container handle drag events */
    transform-origin: center;
  }

  /* Slider */
  .slider-container {
    display: flex;
    align-items: center;
    gap: 12px;
    background: rgba(255, 255, 255, 0.8);
    padding: 12px 16px;
    border-radius: var(--radius-pill);
    box-shadow: inset 0 2px 8px rgba(0,0,0,0.05);
  }

  .zoom-icon { font-size: 1.2rem; }
  
  .zoom-slider {
    flex: 1;
    height: 6px;
    -webkit-appearance: none;
    background: #e0e0e0;
    border-radius: 4px;
    outline: none;
  }
  
  .zoom-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: var(--primary-red);
    cursor: pointer;
    box-shadow: 0 2px 6px rgba(186, 28, 34, 0.4);
  }
</style>
