<script>
  export let idxid = 0;
  export let id = 0;
  export let name = "";
  export let price = "";
  export let quantity = "";
  export let gst = "";
  export let cgst = "";
  export let sgst = "";
  export let amount = "";
  export let handleChange;
  export let removeItem;
</script>

<div class="container">
  <div class="itmbox">
    <label
      ><input
        id="name"
        type="text"
        placeholder="Item Name"
        bind:value={name}
        on:input={(e) => handleChange(idxid, e)}
      /></label
    >
  </div>
  <div class="rowcont itmbox">
    <div class="colcont">
      <label
        ><input
          id="price"
          type="text"
          placeholder="Price"
          bind:value={price}
          on:input={(e) => handleChange(idxid, e)}
        /></label
      >
      <label
        ><input
          id="quantity"
          type="text"
          placeholder="Quantity"
          bind:value={quantity}
          on:input={(e) => handleChange(idxid, e)}
        /></label
      >
    </div>
    <div class="colcont">
      <label
        ><input
          id="gst"
          type="text"
          placeholder="GST (%)"
          bind:value={gst}
          on:input={(e) => handleChange(idxid, e)}
        /></label
      >
      <div class="colcont">
        {#if amount.length > 0 && !isNaN(parseFloat(amount))}
          <span>CGST: ₹{cgst}</span><span>SGST: ₹{sgst}</span>
        {:else}
          <span>CGST: ₹0.00</span><span>SGST: ₹0.00</span>
        {/if}
      </div>
    </div>
  </div>
  <div class="rowcont itmbox">
    <div>
      <span>Amount: </span>
      <span class="boldamt">
        {#if amount.length > 0 && !isNaN(parseFloat(amount))}
          ₹{amount}
        {:else}
          ₹0.00
        {/if}
      </span>
    </div>
    <div>
      <span class="remove" on:click={() => removeItem(idxid)}>Remove Item</span>
    </div>
  </div>
</div>

<style>
  .container {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 1%;
  }

  .colcont {
    display: flex;
    flex-direction: column;
    row-gap: 5px;
  }

  .rowcont {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }

  .itmbox {
    padding: 1%;
  }

  .remove {
    color: #f04438;
    text-decoration-line: underline;
    font-size: 12px;
  }

  .remove:hover {
    cursor: pointer;
  }

  .boldamt {
    font-weight: 600;
    font-size: 18px;
  }
</style>
