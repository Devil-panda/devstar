<script>
  import { onMount } from 'svelte';
  import GIF from 'gif.js.optimized';

  let mode = 'monochrome';
  let gif;
  let previewUrl = '';

  function generateRandomColor(mode) {
    if (mode === 'monochrome') {
      const value = Math.floor(Math.random() * 256);
      return `rgb(${value}, ${value}, ${value})`;
    } else if (mode === 'rgb') {
      const r = Math.floor(Math.random() * 256);
      const g = Math.floor(Math.random() * 256);
      const b = Math.floor(Math.random() * 256);
      return `rgb(${r}, ${g}, ${b})`;
    } else if (mode === 'single-tone') {
      const value = Math.floor(Math.random() * 256);
      return `rgb(${value}, 0, 0)`;
    } else if (mode === 'custom') {
      // Example custom colors (can be customized)
      const colors = ['#FF5733', '#33FF57', '#3357FF', '#FF33A1'];
      return colors[Math.floor(Math.random() * colors.length)];
    }
  }

  function generateRandomGif() {
    gif = new GIF({
      workers: 2,
      quality: 10,
      workerScript: 'node_modules/gif.js.optimized/dist/gif.worker.js' // Reference the worker script correctly
    });

    const width = 200;
    const height = 200;
    const frameCount = 10;

    for (let i = 0; i < frameCount; i++) {
      const canvas = document.createElement('canvas');
      canvas.width = width;
      canvas.height = height;
      const ctx = canvas.getContext('2d');
      ctx.fillStyle = generateRandomColor(mode);
      ctx.fillRect(0, 0, width, height);
      gif.addFrame(canvas, { delay: 200 });
    }

    gif.on('finished', function(blob) {
      const url = URL.createObjectURL(blob);
      previewUrl = url;
    });

    gif.render();
  }

  function downloadGif() {
    if (previewUrl) {
      const a = document.createElement('a');
      a.href = previewUrl;
      a.download = 'random.gif';
      a.click();
    }
  }
</script>

<style>
  .controls {
    margin-bottom: 10px;
  }
  .preview {
    margin-top: 20px;
  }
</style>

<div class="controls">
  <label style="background-color:#374151; padding:2px 2px; color:white">
    Randomization Mode:
    <select bind:value={mode}>
      <option style="background-color:#374151;  color:white" value="monochrome">Monochrome Randomness</option>
      <option style="background-color:#374151;  color:white" value="rgb">RGB Randomness</option>
      <option style="background-color:#374151;  color:white" value="single-tone">Single Color Tone Randomness</option>
      <option style="background-color:#374151;  color:white" value="custom">Custom Color Randomness</option>
    </select>
  </label>
  <button style="background-color:#374151; padding:2px 2px; color:white" on:click={generateRandomGif}>Generate GIF</button>
  <button style="background-color:#374151; padding:2px 2px; color:white" on:click={downloadGif}>Download GIF</button>
</div>

{#if previewUrl}
  <div class="preview">
    <h3 style="background-color:#374151;  color:white ;width:29.5%">Preview:</h3>
    <img src={previewUrl} alt="Generated GIF preview">
  </div>
{/if}
