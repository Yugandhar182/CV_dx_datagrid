<script>
    export let cvData;
    export let onClose;
  
    function handleClose() {
      onClose();
    }
  </script>
  
  <div class="popup-overlay">
    <div class="popup-content">
      <h3>CV View</h3>
      {#if cvData.html}
        <div>
          <div>{@html cvData.html}</div>
          <button on:click={handleClose} class="btn btn-secondary">Close</button>
        </div>
      {:else}
        <p>CV file not found.</p>
        <button on:click={handleClose} class="btn btn-secondary">Close</button>
      {/if}
    </div>
  </div>
  
  <style>
    /* Add your custom styling for the popup overlay and content here */
  </style>
  