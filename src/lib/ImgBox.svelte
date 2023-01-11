<script>
  import FaFileUpload from "svelte-icons/fa/FaFileUpload.svelte";
  import FaTimesCircle from "svelte-icons/fa/FaTimesCircle.svelte";

  let uploaded = false,
    fileinput,
    imageURI;

  const onFileSelected = (e) => {
    let image = e.target.files[0];
    let reader = new FileReader();
    reader.readAsDataURL(image);
    reader.onload = (e) => {
      uploaded = true;
      imageURI = e.target.result;
    };
  };

  const handleClose = () => {
    uploaded = false;
  };
</script>

{#if uploaded}
  <div class="imageBox iconParen">
    <img class="img" src={imageURI} alt="logo" />
    <div class="rightCircle" on:click={handleClose}><FaTimesCircle /></div>
  </div>
{:else}
  <div
    class="imageBox flexbox"
    on:click={() => {
      fileinput.click();
    }}
  >
    <div class="logo"><FaFileUpload /></div>
    <span>Upload Logo</span>
  </div>
{/if}

<input
  style="display:none"
  type="file"
  accept=".jpg, .jpeg, .png"
  on:change={(e) => onFileSelected(e)}
  bind:this={fileinput}
/>

<style>
  .imageBox {
    height: 125px;
    width: 125px;
    border: 1px solid #b9b9b9;
    border-radius: 15px;
  }

  .iconParen {
    position: relative;
  }

  .rightCircle {
    position: absolute;
    width: 25px;
    height: 25px;
    border-radius: 50%;
    background-color: white;
    right: -7px;
    top: -7px;
    color: #f04438;
  }

  .img {
    height: 100%;
    width: 100%;
    border-radius: 15px;
  }

  .logo {
    height: 30px;
    width: 30px;
  }

  .flexbox {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    row-gap: 10px;
    color: #4c4c4c;
  }
</style>
