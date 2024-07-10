<script>
  let selectedColor = "#ffffff"; // Default color is white
  let gifUrl = ""; // URL of the generated GIF

  // Function to generate the GIF
  const generateGif = () => {
    gifUrl = generateColorGif(selectedColor);
  };

  // Function to handle color selection
  const selectColor = (color) => {
    selectedColor = color;
    generateGif();
  };

  // Function to download the generated GIF
  const downloadGif = () => {
    if (gifUrl) {
      const link = document.createElement('a');
      link.href = gifUrl;
      link.download = 'tiny.gif';
      link.click();
    }
  };

  // Function to generate a color-based 1x1 pixel GIF
  const generateColorGif = (color) => {
    const canvas = document.createElement("canvas");
    canvas.width = 1;
    canvas.height = 1;
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = color;
    ctx.fillRect(0, 0, 1, 1);
    return canvas.toDataURL("image/gif");
  };

  // Generate initial GIF
  generateGif();
</script>

<style>
  .container {
    margin: 20px;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 5px;
    max-width: 400px;
  }

  label {
    margin-bottom: 10px;
    display: block;
  }

  img {
    margin-top: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    width: 150px; /* Adjust the width as needed */
  }

  button {
    margin-top: 10px;
    padding: 10px 20px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
</style>

<div class="container">
  <!-- Color selection label -->
  <label style="background-color:#374151;color:white; width:56%" for="colorPicker">Choose color:</label>
  <!-- Color selection input -->
  <input type="color" id="colorPicker" bind:value={selectedColor} on:input={generateGif}>

  <!-- Preview generated GIF -->
  {#if gifUrl}
    <div>
      <h3 style="background-color:#374151;color:white; width:35%">Preview:</h3>
      <img src={gifUrl} alt="Generated GIF">
    </div>
  {/if}

  <!-- Download button -->
  <button on:click={downloadGif} disabled={!gifUrl}>Download GIF</button>
</div>
