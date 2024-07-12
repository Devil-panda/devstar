<script>
    import { onMount } from 'svelte';
    import { GifReader } from 'omggif';
  
    let gifFile = null;
    let gifInfo = {};
    let frameInfos = [];
    let frameNumber = 1;
    let gifUrl = '';
    let globalColorTable = [];
    
    let showGeneralGifInfo = true;
    let showGlobalColorTable = false;
    let showFrameInfo = false;
    
    const handleFileSelect = (event) => {
      const file = event.target.files[0];
      if (file && file.type === "image/gif") {
        gifFile = file;
        gifUrl = URL.createObjectURL(file);
        clearPreviousData();
        extractGifInfo(file);
      }
    };
  
    const clearPreviousData = () => {
      gifInfo = {};
      frameInfos = [];
      globalColorTable = [];
      frameNumber = 1;
    };
  
    const extractGifInfo = (file) => {
      const reader = new FileReader();
      reader.onload = (e) => {
        const arrayBuffer = e.target.result;
        if (arrayBuffer instanceof ArrayBuffer) {
          const gifReader = new GifReader(new Uint8Array(arrayBuffer));
          gifInfo = {
            width: gifReader.width,
            height: gifReader.height,
            numFrames: gifReader.numFrames(),
          };
          extractFrameInfos(gifReader);
          extractGlobalColorTable(arrayBuffer);
        }
      };
      reader.readAsArrayBuffer(file);
    };
  
    const extractFrameInfos = (gifReader) => {
      const numFrames = gifReader.numFrames();
      for (let i = 0; i < numFrames; i++) {
        frameInfos.push({
          frameNumber: i + 1,
          delay: gifReader.frameInfo(i).delay,
        });
      }
    };
  
    const extractGlobalColorTable = (arrayBuffer) => {
      const uint8View = new Uint8Array(arrayBuffer);
      const gctFlag = (uint8View[10] & 0x80) >> 7;
      const gctSize = uint8View[10] & 0x07;
      if (gctFlag) {
        const colorTableSize = 2 ** (gctSize + 1);
        for (let i = 0; i < colorTableSize; i++) {
          const offset = 13 + i * 3;
          globalColorTable.push({
            red: uint8View[offset],
            green: uint8View[offset + 1],
            blue: uint8View[offset + 2]
          });
        }
      }
    };
  
    const formatGifInfo = () => {
      let info = `GIF Information:\n`;
      if (showGeneralGifInfo) {
        info += `Width: ${gifInfo.width} px\n`;
        info += `Height: ${gifInfo.height} px\n`;
        info += `Number of Frames: ${gifInfo.numFrames}\n`;
      }
      if (showFrameInfo) {
        info += `Frame Information:\n`;
        frameInfos.forEach(frameInfo => {
          info += `Frame Number: ${frameInfo.frameNumber}\n`;
          info += `Frame Delay: ${frameInfo.delay} ms\n`;
        });
      }
      if (showGlobalColorTable) {
        info += `Global Color Table:\n`;
        globalColorTable.forEach((color, index) => {
          info += `Color ${index + 1}: rgb(${color.red}, ${color.green}, ${color.blue})\n`;
        });
      }
      return info;
    };
  
    const downloadGifInfo = () => {
      const info = formatGifInfo();
      const blob = new Blob([info], { type: 'text/plain' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'gif-info.txt';
      a.click();
      URL.revokeObjectURL(url);
    };
  </script>
  
  <div class="container">
    <div class="title">Analyze GIF Info</div>
    <div class="input">
      <input type="file" accept="image/gif" on:change={handleFileSelect} id="gif-upload" style="display: none;" />
      <label for="gif-upload" class="upload-button">Upload GIF</label>
      <div class="options">
        <h3 class="text-lg font-bold">Tool Options</h3>
        <h4 class="text-base font-medium underline">General GIF Info</h4>
        <input type="checkbox" id="generalGifInfo" bind:checked={showGeneralGifInfo}>
        <label for="generalGifInfo" class="text-sm">Print GIF Information</label><br>
        <input type="checkbox" id="globalColorTable" bind:checked={showGlobalColorTable}>
        <label for="globalColorTable" class="text-sm">Print Global Color Table</label><br>
        
        <h4 class="text-base font-medium underline">Frame Info</h4>
        <input type="checkbox" id="frameInfo" bind:checked={showFrameInfo}>
        <label for="frameInfo" class="text-sm">Print Frame Information</label><br>
      </div>
      <button class="download-button" on:click={downloadGifInfo}>Download Info</button>
    </div>
    <div class="output">
      {#if gifFile}
        <h2 class="font-bold">GIF Debug Info</h2>
        <div>
          <div class="preview">
            <img src={gifUrl} alt="GIF Preview" />
          </div>
          {#if showGeneralGifInfo}
            <h3 class="font-medium underline decoration-double">General GIF Info</h3>
            <p>Width: {gifInfo.width} px</p>
            <p>Height: {gifInfo.height} px</p>
            <p>Number of Frames: {gifInfo.numFrames}</p>
          {/if}
          {#if showFrameInfo}
            <h3 class="font-medium underline decoration-double">Frame Info</h3>
            {#each frameInfos as frameInfo}
              <p>Frame Number: {frameInfo.frameNumber}</p>
              <p>Frame Delay: {frameInfo.delay} ms</p>
            {/each}
          {/if}
          {#if showGlobalColorTable}
            <h3 class="font-medium underline decoration-double">Global Color Table</h3>
            <div class="color-table">
              {#each globalColorTable as color}
                <div class="color-swatch" style="background-color: rgb({color.red}, {color.green}, {color.blue});"></div>
              {/each}
            </div>
          {/if}
        </div>
      {/if}
    </div>
  </div>
  
  <style>
    .container {
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      padding: 20px;
      align-items: center;
      max-width: 900px;
      margin: auto;
      text-align: center;
      color: white;
    }
  
    .title {
      font-size: 24px;
      margin-bottom: 20px;
      font-weight: 600;
      color: #ff6347;
    }
  
    .upload-button, .download-button {
      display: inline-block;
      padding: 12px 24px;
      background-color: #28a745;
      color: white;
      text-decoration: none;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }
  
    .upload-button:hover {
      background-color: #218838;
    }
  
    .download-button {
      background-color: #007bff;
    }
  
    .download-button:hover {
      background-color: #0056b3;
    }
  
    .output {
      height: 200px;
      width: 100%;
      overflow: auto;
      color: #ff6347;
    }
  
    .input {
      color: #ff6347;
    }
  
    .options {
      margin-top: 20px;
    }
  
    .preview {
      margin-top: 20px;
    }
  
    .color-table {
      display: flex;
      flex-wrap: wrap;
    }
  
    .color-swatch {
      width: 20px;
      height: 20px;
      margin: 1px;
    }
  </style>
  