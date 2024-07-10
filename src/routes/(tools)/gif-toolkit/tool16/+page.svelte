<script>
    import { onMount } from 'svelte';
    import { parseGIF, decompressFrames } from 'gifuct-js';
    import { saveAs } from 'file-saver';
  
    let file;
    let frameNumbers = '';
    let extractedFrames = [];
  
    const extractFrames = () => {
      if (!file || !frameNumbers) {
        alert('Please upload a GIF and input frame numbers.');
        return;
      }
  
      const frameIndices = frameNumbers.split(',').map(num => parseInt(num.trim(), 10));
      const reader = new FileReader();
  
      reader.onload = async (e) => {
        const buffer = e.target.result;
        const gif = parseGIF(buffer);
        const frames = decompressFrames(gif, true);
  
        const baseCanvas = document.createElement('canvas');
        const baseCtx = baseCanvas.getContext('2d');
  
        baseCanvas.width = frames[0].dims.width;
        baseCanvas.height = frames[0].dims.height;
  
        const coalescedFrames = frames.map((frame, index) => {
          if (index === 0) {
            baseCtx.putImageData(new ImageData(new Uint8ClampedArray(frame.patch), frame.dims.width, frame.dims.height), 0, 0);
          } else {
            baseCtx.clearRect(0, 0, frame.dims.width, frame.dims.height);
            baseCtx.putImageData(new ImageData(new Uint8ClampedArray(frame.patch), frame.dims.width, frame.dims.height), frame.dims.left, frame.dims.top);
          }
          return baseCanvas.toDataURL();
        });
  
        extractedFrames = frameIndices.map(index => coalescedFrames[index]).filter(frame => frame !== null);
      };
  
      reader.readAsArrayBuffer(file);
    };
  
    const handleDownload = () => {
      extractedFrames.forEach((frame, index) => {
        const link = document.createElement('a');
        link.href = frame;
        link.download = `frame${index + 1}.png`;
        link.click();
      });
    };
  
    onMount(() => {
      // Cleanup URL object on component destroy
      return () => {
        if (file) {
          URL.revokeObjectURL(file);
        }
      };
    });
  </script>
  
  <style>
    .preview {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
    }
    .preview img {
      max-width: 100px;
      max-height: 100px;
    }
  </style>
  
  <div>
    <input type="file" accept="image/gif" on:change="{e => file = e.target.files[0]}" />
    <div>
        <input type="text" placeholder="Enter frame numbers (comma-separated)" bind:value="{frameNumbers}" />
    </div>
    <button style="background-color:#374151;color:white;padding:5px 6px 5px 12px;margin:0px 16px 0px -16px" on:click="{extractFrames}">Extract Frames</button>
    <button style="background-color:#374151;color:white;padding:5px 6px 5px 12px; margin:0px 16px 0px -16px" on:click="{handleDownload}">Download Frames</button>
     <div style="padding:2px"></div>
    <div class="preview">
      {#each extractedFrames as frame}
        <img src="{frame}" alt="Extracted Frame" />
      {/each}
    </div>
  </div>
  