<script>
  import { saveAs } from 'file-saver';
  import { parseGIF, decompressFrames } from 'gifuct-js';
  import gifshot from 'gifshot';
  let gifFile;
  let borderedGifBlob;
  let borderWidth = 5;
  let gifURL;
  let originalGif;
  let downloadLink;
  let borderColor = '#FF0000';

  async function handleFileUpload(event) {
      const file = event.target.files[0];
      if (file && file.type === 'image/gif') {
          gifFile = file;
          gifURL = URL.createObjectURL(file);
          originalGif = gifURL;
          await addBorderToGif();
      } else {
          alert('Please upload a valid GIF file.');
      }
  }

  async function addBorderToGif() {
      if (!gifFile) return;
      const frames = await extractFrames(gifFile);
      const borderedFrames = await addBorderToFrames(frames, borderWidth, borderColor);
      borderedGifBlob = await createBorderedGif(borderedFrames, frames[0].dims.width, frames[0].dims.height);
  }

  function downloadBorderedGif() {
      if (borderedGifBlob) {
          saveAs(borderedGifBlob, 'bordered.gif');
      }
  }

  async function extractFrames(file) {
      return new Promise((resolve) => {
          const reader = new FileReader();
          reader.onload = async (e) => {
              const gif = parseGIF(e.target.result);
              const frames = decompressFrames(gif, true);
              resolve(frames);
          };
          reader.readAsArrayBuffer(file);
      });
  }

  async function addBorderToFrames(frames, borderWidth, borderColor) {
      return Promise.all(frames.map(async frame => {
          const canvas = document.createElement('canvas');
          const ctx = canvas.getContext('2d');
          canvas.width = frame.dims.width + 2 * borderWidth;
          canvas.height = frame.dims.height + 2 * borderWidth;
          ctx.fillStyle = borderColor;
          ctx.fillRect(0, 0, canvas.width, canvas.height);
          const imageData = new ImageData(frame.patch, frame.dims.width, frame.dims.height);
          ctx.putImageData(imageData, borderWidth, borderWidth);
          return new Promise((resolve) => {
              canvas.toBlob((blob) => {
                  const img = new Image();
                  img.src = URL.createObjectURL(blob);
                  img.onload = () => resolve(img.src);
              }, 'image/png');
          });
      }));
  }

  async function createBorderedGif(images, width, height) {
      return new Promise((resolve) => {
          gifshot.createGIF({
              images,
              gifWidth: width + 2 * borderWidth,
              gifHeight: height + 2 * borderWidth,
              interval: 0.1,
              numFrames: images.length,
              frameDuration: 1,
              sampleInterval: 10,
              numWorkers: 2
          }, function (obj) {
              if (!obj.error) {
                  const blob = dataURLToBlob(obj.image);
                  resolve(blob);
              }
          });
      });
  }

  function dataURLToBlob(dataURL) {
      const byteString = atob(dataURL.split(',')[1]);
      const ab = new ArrayBuffer(byteString.length);
      const ia = new Uint8Array(ab);
      for (let i = 0; i < byteString.length; i++) {
          ia[i] = byteString.charCodeAt(i);
      }
      return new Blob([ab], { type: 'image/gif' });
  }
</script>

<div class="container">
  <div class="title">Add Border to GIF</div>
  <div class="upload-container">
      <input type="file" accept="image/gif" on:change={handleFileUpload} id="gif-upload" style="display: none;">
      <label for="gif-upload" class="upload-button">Upload GIF</label>
  </div>
  {#if gifFile}
      <div class="control-container">
          <label for="borderWidth">Border Width: </label>
          <input type="number" id="borderWidth" min="0" max="50" bind:value={borderWidth} on:input={addBorderToGif} />
          <label for="borderColor">Border Color:</label>
          <input type="color" id="borderColor" bind:value={borderColor} on:input={addBorderToGif} />
      </div>
      <div class="preview-container">
          <div class="preview">
              <div>Original GIF</div>
              <img src={originalGif} alt="Original GIF">
          </div>
          <div class="preview">
              <div>Bordered GIF</div>
              {#if borderedGifBlob}
                  <img src={URL.createObjectURL(borderedGifBlob)} alt="Bordered GIF Preview">
              {/if}
          </div>
      </div>
      <div>
          <a bind:this={downloadLink} class="download-button" on:click={downloadBorderedGif}>Download Bordered GIF</a>
      </div>
  {/if}
</div>

<style>
  .container {
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
  .upload-container {
      margin-bottom: 20px;
  }

  .upload-button {
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

  .preview-container {
      display: flex;
      justify-content: space-around;
      margin-top: 20px;
      gap: 20px;
      font-size: 18px;
      color: #ff6347;
  }

  .preview {
      width: 45%;
      text-align: center;
  }

  .preview img,
  .preview canvas {
      max-width: 100%;
      height: auto;
      border: 2px solid #ccc;
      border-radius: 5px;
      margin-top: 10px;
  }

  .control-container {
      margin: 20px 0;
      color: #ff6347;
  }

  .control-container label {
      font-weight: 600;
      margin-right: 10px;
  }

  .download-button {
      display: inline-block;
      margin-top: 20px;
      padding: 12px 24px;
      background-color: #007bff;
      color: white;
      text-decoration: none;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s ease;
  }

  .download-button:hover {
      background-color: #0056b3;
  }
</style>

