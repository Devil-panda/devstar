<script>
  import { parseGIF, decompressFrames } from 'gifuct-js';
  import GIF from 'gif.js.optimized';

  let file;
  let gifData;
  let frames = [];
  let modifiedGifUrl;
  let startFrame = 0;
  let endFrame = 0;

  const handleFileUpload = async (event) => {
    const uploadedFile = event.target.files[0];
    if (uploadedFile) {
      const arrayBuffer = await uploadedFile.arrayBuffer();
      gifData = parseGIF(arrayBuffer);
      frames = decompressFrames(gifData, true);
      console.log('Frames extracted:', frames.length);
    }
  };

  const createImageFromFrame = (frame) => {
    const canvas = document.createElement('canvas');
    canvas.width = frame.dims.width;
    canvas.height = frame.dims.height;
    const ctx = canvas.getContext('2d', { willReadFrequently: true });
    const imageData = new ImageData(new Uint8ClampedArray(frame.patch), frame.dims.width, frame.dims.height);
    ctx.putImageData(imageData, 0, 0);
    return canvas;
  };

  const removeFramesAndGenerateGif = () => {
    if (!frames.length) {
      alert('Please upload a GIF.');
      return;
    }

    if (startFrame === undefined || endFrame === undefined) {
      alert('Please enter a valid frame range.');
      return;
    }

    if (startFrame < 0 || endFrame < 0) {
      alert('Frame numbers must be non-negative.');
      return;
    }

    if (startFrame >= endFrame) {
      alert('Start frame must be less than end frame.');
      return;
    }

    if (endFrame >= frames.length) {
      alert('End frame must be less than the total number of frames.');
      return;
    }

    const filteredFrames = frames.filter((_, index) => index < startFrame || index > endFrame);
    const gif = new GIF({ workers: 2, quality: 10, workerScript: 'node_modules/gif.js.optimized/dist/gif.worker.js' });

    filteredFrames.forEach(frame => {
      const image = createImageFromFrame(frame);
      gif.addFrame(image, { delay: frame.delay });
    });

    gif.on('finished', (blob) => {
      modifiedGifUrl = URL.createObjectURL(blob);
    });

    gif.render();
  };

  const downloadGif = () => {
    if (modifiedGifUrl) {
      const link = document.createElement('a');
      link.href = modifiedGifUrl;
      link.download = 'modified.gif';
      link.click();
    }
  };
</script>

<style>
  .preview {
    max-width: 300px;
    max-height: 300px;
    margin-top: 10px;
  }
  
  .frame-inputs {
    display: flex;
    gap: 10px;
    margin-top: 10px;
  }
  
  .frame-inputs label {
    display: flex;
    flex-direction: column;
    background-color: #374151;
    color: white;
    padding: 5px;
    border-radius: 5px;
  }

  .controls {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 10px;
  }

  .controls button {
    background-color: #374151;
    color: white;
    padding: 10px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
</style>

<div>
  <input type="file" accept="image/gif" on:change={handleFileUpload} />
  <div class="frame-inputs">
    <label>
      Start Frame:
      <input style="color:black" type="number" bind:value={startFrame} min="0" />
    </label>
    <label>
      End Frame:
      <input style="color:black" type="number" bind:value={endFrame} min="0" />
    </label>
  </div>
  <div class="controls">
    <button on:click={removeFramesAndGenerateGif}>Remove Frames</button>
    <div style="padding:2px"></div>
    {#if modifiedGifUrl}
      <div>
        <h3 style="background-color:#374151; color:white">Preview</h3>
        <img class="preview" src={modifiedGifUrl} alt="Modified GIF" />
      </div>
      <button on:click={downloadGif}>Download Modified GIF</button>
    {/if}
  </div>
</div>
