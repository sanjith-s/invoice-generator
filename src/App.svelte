<script>
  // @ts-nocheck

  import { HtmlTag } from "svelte/internal";
  import ImgBox from "./lib/ImgBox.svelte";
  import Item from "./lib/Item.svelte";
  import { jsPDF } from "jspdf";
  import 'jspdf-autotable';


  let imgData="";

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
    contact: "",
    gstin: "",
  };

  let billerDetails = {
    name: "",
    email: "",
    addr1: "",
    addr2: "",
    city: "",
    zipcode: "",
    contact: "",
    gstin: "",
  };

  let invoiceDetails = {
    number: "",
    date: "",
    term: "",
    notes: "",
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

  let paymentDetails = {
    total: "",
    paid: "",
    balance: "",
  };

  let items = [{...itemObj}];

  let consolidatedPrice = {
    subtotal: 0,
    cgst: 0,
    sgst: 0,
    total: 0,
  };

  const generatePDF = () => {
    generateInvoice();
  };

  const handleChange = (id, e) => {
    //console.log(id, e.target.id, e.target.value);
    let nameKey = e.target.id;

    let value = e.target.value;
    if (nameKey == "gst") {
      if (parseFloat(e.target.value) > 100) value = "100";
      else if (parseFloat(e.target.value) < 0) value = "0";
    }
    items[id][nameKey] = value;

    //console.log(items);

    let item = items[id];
    let price = parseFloat(item.price);
    let quantity = parseFloat(item.quantity);
    let gst = parseFloat(item.gst);

    if (isNaN(price) || isNaN(quantity) || isNaN(gst)) {
      return;
    }

    let gstamt = price * quantity * (gst / 100);

    item.cgst = (gstamt / 2).toFixed(2);
    item.sgst = (gstamt / 2).toFixed(2);
    item.amount = (price * quantity * (1 + gst / 100)).toFixed(2);
    items[id] = { ...item };

    let subtotal = 0;
    let totalCgst = 0;
    let totalSgst = 0;
    let total = 0;

    for (let item of items) {
      let itmTot = parseFloat(item.amount);
      let itmCgst = parseFloat(item.cgst);
      let itmSgst = parseFloat(item.sgst);

      subtotal += itmTot - (itmCgst + itmSgst);
      totalCgst += itmCgst;
      totalSgst += itmSgst;
      total += itmTot;
    }

    consolidatedPrice.subtotal = subtotal;
    consolidatedPrice.cgst = totalCgst;
    consolidatedPrice.sgst = totalSgst;
    consolidatedPrice.total = total;

    //console.log(consolidatedPrice);
  };

  const addItem = () => {
    let newItem = { ...itemObj };
    newItem.id = items[items.length - 1].id + 1;
    items = [...items, newItem];
    //console.log(items);
  };

  const removeItem = (id) => {
    if (items.length == 1) return;

    let remItem = items.splice(id, 1)[0];
    items = [...items];

    consolidatedPrice.subtotal -=
      parseFloat(remItem.amount) -
      (parseFloat(remItem.cgst) + parseFloat(remItem.sgst));
    consolidatedPrice.cgst -= parseFloat(remItem.cgst);
    consolidatedPrice.sgst -= parseFloat(remItem.sgst);
    consolidatedPrice.total -= parseFloat(remItem.amount);

    //console.log(items);
  };

  function calcPayment() {
    var t = consolidatedPrice.total;
    var total = document.getElementById("totalamt");
    var pamt = document.getElementById("pamt");
    var a = pamt.value;

    var bal = +t - +a;
    var balance = document.getElementById("pbalance");
    balance.innerText = "???" + +bal;
    //console.log(total);
  }

  var fontSizes = {
    HeadTitleFontSize: 18,
    Head2TitleFontSize: 16,
    TitleFontSize: 14,
    SubTitleFontSize: 12,
    NormalFontSize: 10,
    SmallFontSize: 8,
  };
  var lineSpacing = {
    NormalSpacing: 14,
  };

  var logoDim = {
    w: 75,
    h: 75,
  };

  function generateInvoice() {
    var doc = new jsPDF("p", "pt");
    var rightStartCol1 = 400;
    var rightStartCol2 = 480;
    var InitialstartX = 40;
    var startX = 40;
    var InitialstartY = 50;
    var startY = 0;
    var lineHeights = 12;
    doc.setFontSize(fontSizes.SubTitleFontSize);
    doc.setFont("helvetica", "bold");
    if(imgData.length>0) {
      doc.addImage(
      imgData,
      "JPEG",
      startX,
      (startY += 50),
      logoDim.w,
      logoDim.h
    );
    } else {
      startY += 50;
    }
    

    var tempY = InitialstartY;
    doc.setFont("helvetica", "bold");
    doc.text(
      "Invoice No: ",
      rightStartCol1,
      (tempY += lineSpacing.NormalSpacing)
    );
    doc.setFont("helvetica", "normal");
    doc.text(invoiceDetails.number, rightStartCol2, tempY);
    doc.setFont("helvetica", "bold");
    doc.text(
      "Invoice Date: ",
      rightStartCol1,
      (tempY += lineSpacing.NormalSpacing)
    );
    doc.setFont("helvetica", "normal");
    doc.text(invoiceDetails.date, rightStartCol2, tempY);

    startY += (logoDim.h+20);

    doc.setFontSize(fontSizes.Head2TitleFontSize);
    doc.setFont("helvetica", "bold");

    var width = doc.internal.pageSize.getWidth()
    doc.text("TAX INVOICE", width/2, (startY += lineSpacing.NormalSpacing + 2), {align: "center"});

    doc.setFontSize(fontSizes.NormalFontSize);
    doc.setFont("helvetica", "bold");

    startY += 20;


    //-------Customer Info Billing---------------------
    var startBilling = startY;

    doc.text("Biller", startX, (startY += lineSpacing.NormalSpacing));
    doc.text(
      billerDetails.name,
      startX,
      (startY += lineSpacing.NormalSpacing)
    );
    doc.setFontSize(fontSizes.NormalFontSize);
    doc.text("GSTIN ", startX, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(billerDetails.gstin, 80, startY);
    doc.setFont("helvetica", "bold");
    doc.text("Address ", startX, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(billerDetails.addr1, 80, startY);
    doc.text(
      billerDetails.addr2,
      80,
      (startY += lineSpacing.NormalSpacing)
    );
    doc.setFont("helvetica", "bold");
    doc.text("City ", startX, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(billerDetails.city, 80, startY);
    doc.setFont("helvetica", "bold");
    doc.text("PIN ", startX, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(billerDetails.zipcode, 80, startY);
    doc.setFont("helvetica", "bold");
    doc.text("Email ", startX, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(billerDetails.email, 80, startY);
    doc.setFont("helvetica", "bold");
    doc.text("Phone ", startX, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(billerDetails.contact, 80, startY);

    //-------Customer Info Shipping---------------------
    var rightcol1 = 340;
    var rightcol2 = 400;
    startY = startBilling;
    doc.setFont("helvetica", "bold");
    doc.text(
      "Client",
      rightcol1,
      (startY += lineSpacing.NormalSpacing)
    );
    doc.text(
      clientDetails.name,
      rightcol1,
      (startY += lineSpacing.NormalSpacing)
    );
    doc.setFontSize(fontSizes.NormalFontSize);
    doc.setFont("helvetica", "bold");
    doc.text("GSTIN ", rightcol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(clientDetails.gstin, rightcol2, startY);
    doc.setFont("helvetica", "bold");
    doc.text("Address ", rightcol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(clientDetails.addr1, rightcol2, startY);
    doc.text(
      clientDetails.addr2,
      rightcol2,
      (startY += lineSpacing.NormalSpacing)
    );
    doc.setFont("helvetica", "bold");
    doc.text("City ", rightcol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(clientDetails.city, rightcol2, startY);

    doc.setFont("helvetica", "bold");
    doc.text("PIN ", rightcol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(clientDetails.zipcode, rightcol2, startY);

    doc.setFont("helvetica", "bold");
    doc.text("Email ", rightcol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(clientDetails.email, rightcol2, startY);

    doc.setFont("helvetica", "bold");
    doc.text("Phone ", rightcol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(clientDetails.contact, rightcol2, startY);

    doc.setFontSize(8);
    doc.setFont("helvetica", "normal");


    let bodyData = [];
    let i = 1;
    for (let obj of items) {
      delete obj.id;
      let objData = Object.values(obj);
      bodyData.push([i, ...objData]);
      i++;
    }

    doc.autoTable({
    head: [
      ['ID', 'Item Name', 'Price', 'Quantity', 'GST(%)', 'CGST', 'SGST', 'Amount'],
    ],
    body: bodyData,
      theme: 'grid',
      margin: {
        top: 50,
      },
      startY: (startY += 50),
  });

    //-------Invoice Footer---------------------
    var rightcol1 = 340;
    var rightcol2 = 430;

    console.log(doc.lastAutoTable.getWidth);

    startY = doc.lastAutoTable.finalY + 30;
    doc.setFontSize(fontSizes.NormalFontSize);

    doc.setFont("helvetica", "bold");
    doc.text("Sub Total,", rightStartCol1, (startY += lineSpacing.NormalSpacing));
    doc.text(consolidatedPrice.subtotal.toFixed(2), rightStartCol2, startY);
    doc.setFontSize(fontSizes.NormalFontSize);

    doc.setFont("helvetica", "bold");
    doc.text("CGST Rs.", rightStartCol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(consolidatedPrice.cgst.toFixed(2), rightStartCol2, startY);

    doc.setFont("helvetica", "bold");
    doc.text("SGST Rs.", rightStartCol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text(consolidatedPrice.sgst.toFixed(2), rightStartCol2, startY);

    doc.setFont("helvetica", "bold");
    doc.text("Total GST Rs.", rightStartCol1, (startY += lineSpacing.NormalSpacing));
    doc.setFont("helvetica", "normal");
    doc.text((consolidatedPrice.cgst+consolidatedPrice.sgst).toFixed(2), rightStartCol2, startY);

    doc.setFont("helvetica", "bold");
    doc.text(
      "Grand Total Rs.",
      rightStartCol1,
      (startY += lineSpacing.NormalSpacing)
    );
    doc.setFont("helvetica", "normal");
    doc.text(consolidatedPrice.total.toFixed(2), rightStartCol2, startY);
    
    
    startY += 20;

    doc.setFillColor(232, 250, 243);
    doc.rect(startX, startY, startX+rightStartCol2, 100, 'F');


    doc.setTextColor("#000000");

    startY+=5;
    doc.setFont("helvetica", "bold");
    doc.text("Notes", startX+10, (startY +=lineSpacing.NormalSpacing));

    doc.setFont("helvetica", "normal");
    doc.text(invoiceDetails.term, startX+10, startY+lineSpacing.NormalSpacing)

    doc.save(`invoice_${invoiceDetails.number}.pdf`);
  }

  const storeImg = (data) => {
    imgData = data;
  }
</script>

<div class="container bg-white">
  <div class="py-5 text-center">
    <img
      class="d-block mx-auto mb-4"
      src="./public/invoice.png"
      alt=""
      width="80"
    />
    <h1>Invoice Generator</h1>
    <p>Get your invoice generated through our app for free!!</p>
  </div>
  <hr />
  <form>
    <div class="invoiceBox row g-5" id="invoice">
      <div id="titleBox">
        <span class="title1">Tax Invoice</span>
        <ImgBox {storeImg}/>
      </div>

      <div class="row g-5">
        <div id="billerDetails" class="details col-lg-6 col-md-12">
          <h3 class="mb-3">From</h3>
          <label class="form-label">
            <input
              id="bname"
              type="text"
              placeholder="Name"
              class="form-control"
              bind:value={billerDetails.name}
            /></label
          >
          <br />
          <label class="form-label">
            <input
              id="bemail"
              type="text"
              placeholder="Email"
              class="form-control"
              bind:value={billerDetails.email}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="baddr1"
              type="text"
              placeholder="Address line 1"
              class="form-control"
              bind:value={billerDetails.addr1}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="baddr2"
              type="text"
              placeholder="Address line 2"
              class="form-control"
              bind:value={billerDetails.addr2}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="bcity"
              type="text"
              placeholder="City"
              class="form-control"
              bind:value={billerDetails.city}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="bzip"
              type="text"
              placeholder="Pincode"
              class="form-control"
              bind:value={billerDetails.zipcode}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="bcontact"
              type="text"
              placeholder="Contact No"
              class="form-control"
              bind:value={billerDetails.contact}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="bgstin"
              type="text"
              placeholder="GSTIN"
              class="form-control"
              bind:value={billerDetails.gstin}
            /></label
          >
        </div>

        <div id="clientDetails" class="details col-lg-6 col-md-12">
          <h3 class="mb-3">To</h3>
          <label class="form-label"
            ><input
              id="cname"
              type="text"
              placeholder="Name"
              class="form-control"
              bind:value={clientDetails.name}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="cemail"
              type="text"
              placeholder="Email"
              class="form-control"
              bind:value={clientDetails.email}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="caddr1"
              type="text"
              placeholder="Address line 1"
              class="form-control"
              bind:value={clientDetails.addr1}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="caddr2"
              type="text"
              placeholder="Address line 2"
              class="form-control"
              bind:value={clientDetails.addr2}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="ccity"
              type="text"
              placeholder="City"
              class="form-control"
              bind:value={clientDetails.city}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="czip"
              type="text"
              placeholder="Pincode"
              class="form-control"
              bind:value={clientDetails.zipcode}
            /></label
          >
          <br />
          <label class="form-label"
            ><input
              id="ccontact"
              type="text"
              placeholder="Contact No"
              class="form-control"
              bind:value={clientDetails.contact}
            /></label
          >
          <br/>
          <label class="form-label"
            ><input
              id="bgstin"
              type="text"
              placeholder="GSTIN"
              class="form-control"
              bind:value={clientDetails.gstin}
            /></label
          >
        </div>
      </div>

      <div class="row g-5">
        <div id="invoiceDetails">
          <h3 class="mb-3">Invoice Details</h3>
          <label class="form-label col-lg-6"
            >Invoice No<br /><input
              id="inumber"
              class="form-control"
              type="text"
              bind:value={invoiceDetails.number}
            /></label
          >
          <label class="form-label col-lg-6"
            >Invoice Date<br /><input
              id="idate"
              type="date"
              class="form-control"
              bind:value={invoiceDetails.date}
            /></label
          >
          <label class="form-label col-lg-6 "
            >Invoice Term<br /><input
              id="iterm"
              type="text"
              class="form-control"
              bind:value={invoiceDetails.term}
            /></label
          >
        </div>
      </div>

      <div class="row g-5">
        <h3 class="mb-3">Items</h3>
        {#each items as item, id}
          <Item
            class="form-label"
            idxid={id}
            {...item}
            {handleChange}
            {removeItem}
          />
          <hr />
        {/each}
      </div>

      <button type="button" class="add" on:click={addItem}>
        <i class="fa-solid fa-plus" /> Add Item</button
      >

      <div id="totalBox">
        {#if consolidatedPrice.total > 0 && !isNaN(consolidatedPrice.total)}
          <div class="total-box-child">
            Subtotal <span class="total-bold-text"
              >???{consolidatedPrice.subtotal.toFixed(2)}</span
            >
          </div>
          <div class="total-box-child">
            CGST <span class="total-bold-text">???{consolidatedPrice.cgst.toFixed(2)}</span>
          </div>
          <div class="total-box-child">
            SGST <span class="total-bold-text">???{consolidatedPrice.sgst.toFixed(2)}</span>
          </div>
          <div class="total-box-child">
            Total <span class="total-bold-text">???{consolidatedPrice.total.toFixed(2)}</span
            >
          </div>
        {:else}
          <div class="total-box-child">
            Subtotal <span class="total-bold-text">???0.00</span>
          </div>
          <div class="total-box-child">
            CGST <span class="total-bold-text">???0.00</span>
          </div>
          <div class="total-box-child">
            SGST <span class="total-bold-text">???0.00</span>
          </div>
          <div class="total-box-child">
            Total <span id="totalamt" class="total-bold-text">???0.00</span>
          </div>
        {/if}
      </div>

      <!-- <div id="paymentDetails row g-5" class="details">
        <h4 class="mb-3">Payment Details</h4>
        <div class="col-lg-6">
          Total Amount<span id="ptotal" class="total-bold-text"
            >???{consolidatedPrice.total}</span
          >
        </div>
        <div class="row col-lg-6">
          <label class="form-label"
            >Amount Paid: ???<input
              class="pamt form-control"
              id="pamt"
              type="text"
              placeholder="Enter the amount"
              on:keypress={calcPayment}
              on:keyup={calcPayment}
              bind:value={paymentDetails.paid}
            /></label
          >
        </div>
        <div class="col-lg-6">
          Balance Due: <span id="pbalance" class="total-bold-text">???0.00</span>
        </div>
      </div> -->

      <!-- <div>
        <div id="invoiceDetails" class="input-group input-group-lg">
          <label class="text-box-big-label"
            >Notes<br /><input
              id="inotes"
              type="text"
              class="form-control text-box-big"
              bind:value={invoiceDetails.notes}
            /></label
          >
        </div>
      </div> -->


      <button type="button" class="generate" on:click={generatePDF}>
        <i class="fa-regular fa-file" /> Generate Invoice</button
      >
    </div>
  </form>
</div>

<!-- Bootstrap -->
<link
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css"
  rel="stylesheet"
  integrity="sha384-GLhlTQ8iRABdZLl6O3oVMWSktQOp6b7In1Zl3/Jr59b6EGGoI1aFkw7cmDA6j6gD"
  crossorigin="anonymous"
/>

<!-- Iconss -->
<link
  rel="stylesheet"
  href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta2/css/all.min.css"
  integrity="sha512-YWzhKL2whUzgiheMoBFwW8CKV4qpHQAEuvilg9FAn5VJUDwKZZxkJNuGM4XkWuk94WCrrwslk8yWNGmY1EduTA=="
  crossorigin="anonymous"
  referrerpolicy="no-referrer"
/>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Inter&display=swap");
  @import url("https://fonts.googleapis.com/css2?family=Inter:wght@600&display=swap");

  #titleBox {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
  }

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
    padding: 2%;
    font-family: "Inter", sans-serif;
    margin: auto;
    width: 100%;
  }

  .text-box-big {
    height: 100%;
    width: 50%;
  }

  .text-box-big-label {
    height: 100%;
    width: 100%;
  }

  .generate {
    background: #0066ff;
    border-radius: 999px;
    box-shadow: #005de2 0 10px 20px -10px;
    box-sizing: border-box;
    color: #ffffff;
    cursor: pointer;
    font-family: Inter, Helvetica, "Apple Color Emoji", "Segoe UI Emoji",
      NotoColorEmoji, "Noto Color Emoji", "Segoe UI Symbol", "Android Emoji",
      EmojiSymbols, -apple-system, system-ui, "Segoe UI", Roboto,
      "Helvetica Neue", "Noto Sans", sans-serif;
    font-size: 16px;
    font-weight: 700;
    line-height: 24px;
    opacity: 1;
    outline: 0 solid transparent;
    padding: 8px 18px;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
    width: fit-content;
    word-break: break-word;
    border: 0;
    margin: auto;
  }

  .add {
    background-color: #ffffff;
    border: 1px solid #222222;
    border-radius: 8px;
    box-sizing: border-box;
    color: #12b76a;
    cursor: pointer;
    display: inline-block;
    font-family: Circular, -apple-system, BlinkMacSystemFont, Roboto,
      "Helvetica Neue", sans-serif;
    font-size: 16px;
    font-weight: 600;
    line-height: 20px;
    margin: auto;
    outline: none;
    padding: 13px 23px;
    position: relative;
    text-align: center;
    text-decoration: none;
    touch-action: manipulation;
    transition: box-shadow 0.2s, -ms-transform 0.1s, -webkit-transform 0.1s,
      transform 0.1s;
    user-select: none;
    -webkit-user-select: none;
    width: auto;
  }

  .add:focus-visible {
    box-shadow: #14b789 0 0 0 2px, rgba(255, 255, 255, 0.8) 0 0 0 4px;
    transition: box-shadow 0.2s;
  }

  .add:active {
    background-color: #f7f7f7;
    border-color: #14b789;
    transform: scale(0.96);
  }

  .add:disabled {
    border-color: #dddddd;
    color: #dddddd;
    cursor: not-allowed;
    opacity: 1;
  }

  .total-box-child {
    text-align: right;
  }

  .total-bold-text {
    font-weight: 600;
    font-size: 18px;
    margin-left: 1rem;
  }

  .title1 {
    font-size: 48px;
    font-weight: 600;
  }

  /*
  form {
    height: 100%;
    font-family: "Inter", sans-serif;
    font-size: 16px;
    margin: auto;
    width: 100%;
  }

  .grid-container {
    display: grid;
    grid-template-columns: 50% 50%;
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
    width: auto;
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

  .heading h1 {
    text-align: center;
    font-size:30px; font-weight:300; color:#222; letter-spacing:5px;
    text-transform: uppercase;
    color:white;
    display: grid;
    width:100%;
    grid-template-columns: 1fr max-content 1fr;
    grid-template-rows: 20px 0;
    grid-gap: 20px;
    align-items: center;
}

  .heading h1:after,.heading h1:before {
    content: " ";
    display: block;
    border-bottom: 1px solid #000;
    border-top: 1px solid #000;
    height: 4px;
    background-color:#f8f8f8;
}

 
  #titleBox {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }
  
  .total-box-child {
    text-align: right;
  }

  .total-bold-text {
    font-weight: 600;
    font-size: 18px;
    margin-left: 1rem;
  }

  input {
    background: #FFFFFF;
    border: 1px solid #B9B9B9;
    border-radius: 4px;
    padding: .4rem;
    width: 50%;
  }

  .input-width{
    width: 40%;
    margin: 1%;
  }

  .pamt{
    width: 20%;
    font-weight: 600;
    font-size: 18px;
    margin: 0;
    border: #FFFFFF;
  }

  #inotes {
    height: 2.5rem;
  } */
</style>