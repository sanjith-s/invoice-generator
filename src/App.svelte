<script>
  import ImgBox from "./lib/ImgBox.svelte";
  import Item from "./lib/Item.svelte";

  let invoice = {
    number: "",
    date: "",
    customer: "",
    items: [{ description: "", price: "" }],
  };

  let clientDetails = {
    name: "",
    email: "",
    addr1: "",
    addr2: "",
    city: "",
    zipcode: "",
    gstin: "",
  };

  let billerDetails = {
    name: "",
    email: "",
    addr1: "",
    addr2: "",
    city: "",
    zipcode: "",
    gstin: "",
  };

  let itemObj = {
    id: 0,
    name: "",
    price: "",
    quantity: "",
    gst: "",
    cgst: "",
    sgst: "",
    amount: "",
  };

  let items = [itemObj];

  const handleChange = (id, e) => {
    //console.log(id, e.target.id, e.target.value);
    let nameKey = e.target.id;
    items[id][nameKey] = e.target.value;
    console.log(items);

    let item = items[id];
    let price = parseFloat(item.price);
    let quantity = parseFloat(item.quantity);
    let gst = parseFloat(item.gst);

    let gstamt = price * quantity * (gst / 100);
    item.cgst = (gstamt / 2).toFixed(2);
    item.sgst = (gstamt / 2).toFixed(2);
    item.amount = (price * quantity * (1 + gst / 100)).toFixed(2);
    items[id] = { ...item };
  };

  const addItem = () => {
    let newItem = { ...itemObj };
    newItem.id = items[items.length - 1].id + 1;
    items = [...items, newItem];
    console.log(items);
  };

  const removeItem = (id) => {
    items.splice(id, 1);
    items = [...items];
    console.log(items);
  };

  function generateInvoice() {
    // Generate the invoice
  }
</script>

<form>
  <div class="invoiceBox">
    <div id="titleBox">
      <span class="title1">Invoice</span>
      <ImgBox />
    </div>

    <div id="clientDetails" class="details">
      <span class="title2">To</span>
      <label
        ><input
          id="cname"
          type="text"
          placeholder="Name"
          bind:value={clientDetails.name}
        /></label
      >
      <label
        ><input
          id="cemail"
          type="text"
          placeholder="Email"
          bind:value={clientDetails.email}
        /></label
      >
      <label
        ><input
          id="caddr1"
          type="text"
          placeholder="Address line 1"
          bind:value={clientDetails.addr1}
        /></label
      >
      <label
        ><input
          id="caddr2"
          type="text"
          placeholder="Address line 2"
          bind:value={clientDetails.addr2}
        /></label
      >
      <label
        ><input
          id="ccity"
          type="text"
          placeholder="City"
          bind:value={clientDetails.city}
        /></label
      >
      <label
        ><input
          id="czip"
          type="text"
          placeholder="Pincode"
          bind:value={clientDetails.zipcode}
        /></label
      >
      <label
        ><input
          id="cgstin"
          type="text"
          placeholder="GSTIN"
          bind:value={clientDetails.gstin}
        /></label
      >
    </div>

    <div id="billerDetails" class="details">
      <span class="title2">From</span>
      <label
        ><input
          id="bname"
          type="text"
          placeholder="Name"
          bind:value={billerDetails.name}
        /></label
      >
      <label
        ><input
          id="bemail"
          type="text"
          placeholder="Email"
          bind:value={billerDetails.email}
        /></label
      >
      <label
        ><input
          id="baddr1"
          type="text"
          placeholder="Address line 1"
          bind:value={billerDetails.addr1}
        /></label
      >
      <label
        ><input
          id="baddr2"
          type="text"
          placeholder="Address line 2"
          bind:value={billerDetails.addr2}
        /></label
      >
      <label
        ><input
          id="bcity"
          type="text"
          placeholder="City"
          bind:value={billerDetails.city}
        /></label
      >
      <label
        ><input
          id="bzip"
          type="text"
          placeholder="Pincode"
          bind:value={billerDetails.zipcode}
        /></label
      >
      <label
        ><input
          id="bgstin"
          type="text"
          placeholder="GSTIN"
          bind:value={billerDetails.gstin}
        /></label
      >
    </div>
    <div>
      <span class="title2">Items</span>
      {#each items as item, id}
        <Item idxid={id} {...item} {handleChange} {removeItem} />
        <hr />
      {/each}
    </div>
    <button type="button" on:click={addItem}>Add Item</button>
  </div>
</form>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Inter&display=swap");
  @import url("https://fonts.googleapis.com/css2?family=Inter:wght@600&display=swap");

  :global(body) {
    box-sizing: border-box;
    min-height: 100vh;
    margin: 0;
    padding: 0;
    border: 0;
  }

  :global(#app) {
    background-color: #0066ff;
    height: 100%;
    padding: 20%;
    padding-top: 2%;
    padding-bottom: 2%;
  }

  form {
    height: 100%;
    font-family: "Inter", sans-serif;
    font-size: 16px;
    margin: auto;
  }

  .details {
    display: flex;
    flex-direction: column;
    row-gap: 10px;
    justify-content: space-around;
    padding: 10px;
  }

  .invoiceBox {
    background-color: white;
    display: flex;
    flex-direction: column;
    row-gap: 30px;
    width: 100%;
    padding: 20px;
    border-radius: 10px;
  }

  .title2 {
    font-size: 20px;
  }

  .title1 {
    font-size: 48px;
    font-weight: 600;
  }

  #titleBox {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }
</style>
