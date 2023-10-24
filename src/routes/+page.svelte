<script lang="ts">
  import { beforeUpdate, onMount } from "svelte";
  import TrashIcon from "../assets/trashIcon.svelte";
  import InputCount from "../components/InputCount.svelte";
  import {
    moneyInputFormat,
    moneyInputFormatToFloat,
  } from "../utils/inputMoney";
  import Form from "./components/Form.svelte";
  import Header from "./components/Header.svelte";
  import LocalStorageAdapter from "../infra/LocalStorageAdapter";
  import productsMock from "../products-mock.json";

  let productsList: any = [];

  function updateProductsList(newProductsList: any) {
    productsList = newProductsList.detail;
  }

  function handleCheckProduct(index: number) {
    const newProductsList = [...productsList];
    newProductsList[index].isChecked = !newProductsList[index].isChecked;
    productsList = newProductsList;

    const storage = new LocalStorageAdapter();
    storage.set("@ShopList/products", newProductsList);
  }
  function handleMoreQuantity(index: number) {
    const newProductsList = [...productsList];
    newProductsList[index].quantity += 1;
    productsList = newProductsList;

    const storage = new LocalStorageAdapter();
    storage.set("@ShopList/products", newProductsList);
  }
  function handleLessQuantity(index: number) {
    const newProductsList = [...productsList];
    if (newProductsList[index].quantity > 1) {
      newProductsList[index].quantity -= 1;
    }
    productsList = newProductsList;

    const storage = new LocalStorageAdapter();
    storage.set("@ShopList/products", newProductsList);
  }
  function handleChangePrice(e: any, index: number) {
    const newPrice = e.target.value;

    if (newPrice) {
      const newProductsList = [...productsList];
      newProductsList[index].price = `R$ ${moneyInputFormat(newPrice)}`;
      productsList = newProductsList;

      const storage = new LocalStorageAdapter();
      storage.set("@ShopList/products", newProductsList);
    }
  }
  function handleDeleteProduct(index: number) {
    const newProductsList = [...productsList];
    newProductsList.splice(index, 1);
    productsList = newProductsList;

    const storage = new LocalStorageAdapter();
    storage.set("@ShopList/products", newProductsList);
  }

  $: totalPrice = () => {
    const price =
      productsList.length > 0
        ? productsList
            .map((product: any) =>
              product.isChecked
                ? (moneyInputFormatToFloat(product.price) || 0) *
                  product.quantity
                : 0
            )
            .reduce((a: any, b: any) => a + b)
        : 0;

    return `R$ ${moneyInputFormat(String(price.toFixed(2)))}`;
  };

  beforeUpdate(() => {
    const storage = new LocalStorageAdapter();
    const products = storage.get("@ShopList/products");

    if (products) {
      productsList = products;
    } else {
      storage.set("@ShopList/products", productsMock);
      productsList = productsMock;
    }
  });

  onMount(() => {
    let deferredPrompt: any;
    const containerButton = document.getElementById("container-button");
    const addButton = document.getElementById("add-button");
    const cancelButton = document.getElementById("cancel-button");

    console.log(containerButton);

    if (containerButton && addButton && cancelButton) {
      console.log("ueee");
      containerButton.style.display = "none";

      window.addEventListener("beforeinstallprompt", (e) => {
        console.log("service work on beforeinstallprompt");
        // Prevent Chrome 67 and earlier from automatically showing the prompt
        e.preventDefault();
        // Stash the event so it can be triggered later.
        deferredPrompt = e;
        // Update UI to notify the user they can add to home screen
        containerButton.style.display = "flex";

        addButton.addEventListener("click", (e) => {
          // hide our user interface that shows our A2HS button
          containerButton.style.display = "none";
          // Show the prompt
          deferredPrompt.prompt();
          // Wait for the user to respond to the prompt
          deferredPrompt.userChoice.then((choiceResult: any) => {
            if (choiceResult.outcome === "accepted") {
              console.log("User accepted the prompt");
            } else {
              console.log("User dismissed the prompt");
            }
            deferredPrompt = null;
          });
        });
      });

      cancelButton.addEventListener("click", (e) => {
        containerButton.style.display = "none";
      });
    }
  });
</script>

<svelte:head>
  <title>ShopList - Svelte</title>
  <meta name="description" content="Sua lista de compras" />
</svelte:head>

<main>
  <div class="topPage">
    <Header />

    <Form {productsList} on:updateProductsList={updateProductsList} />
  </div>

  {#if productsList.length > 0}
    <div class="productsList">
      {#each productsList as product, index}
        <div class="product" class:checked={product.isChecked}>
          <div class="box">
            <input
              type="checkbox"
              on:change={() => handleCheckProduct(index)}
              checked={product.isChecked}
            />

            <span>{product.name}</span>
          </div>

          <div class="box2">
            <InputCount
              moreQuantity={() => handleMoreQuantity(index)}
              lessQuantity={() => handleLessQuantity(index)}
              bind:quantity={product.quantity}
              isDisabled={product.isChecked}
            />

            <input
              value={product.price}
              style="width: 70px;"
              pattern="[0-9]*"
              inputMode="numeric"
              disabled={product.isChecked}
              on:change={(e) => handleChangePrice(e, index)}
            />

            <!-- svelte-ignore a11y-interactive-supports-focus -->
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <div
              role="button"
              on:click={() =>
                product.isChecked ? null : handleDeleteProduct(index)}
              class="deleteButton"
              class:checked={product.isChecked}
            >
              <TrashIcon />
            </div>
          </div>
        </div>
      {/each}
    </div>
  {:else}
    <div class="emptyList">
      <p>Nenhum produto adicionado</p>
    </div>
  {/if}

  <div class="containerPrice">
    <strong>Total: </strong>
    <span>{totalPrice()}</span>
  </div>

  <div id="container-button" class="addButton">
    <div class="infoButton">
      <img src="/logo.png" alt="logo-shop-list" />
      <span>Instalar ShopList - Svelte</span>
      <div>
        <button id="add-button">Instalar</button>
        <button id="cancel-button" class="buttonVariant"> Cancelar </button>
      </div>
    </div>
  </div>
</main>

<style lang="scss">
  main {
    position: relative;

    margin: 0;
    font-family: "Roboto", sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;

    background-color: #f5f5f5;
  }

  section {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .topPage {
    position: fixed;
    top: 0;
    z-index: 2;
    background-color: #f5f5f5;
    padding-bottom: 10px;
    width: 100%;
  }

  .emptyList {
    max-width: fit-content;
    margin: 40px auto;
    position: relative;
    padding-bottom: 50px;
    padding-top: 250px;
  }

  .productsList {
    max-width: fit-content;
    margin: 40px auto;
    display: flex;
    flex-direction: column;
    gap: 15px;
    position: relative;
    padding-bottom: 50px;
    padding-top: 250px;

    .product {
      display: flex;
      align-items: center;
      gap: 10px;
      justify-content: space-between;
      margin: 0 20px;
      padding-bottom: 15px;

      border-bottom: 1px solid #a1a1aa;

      @media (max-width: 595px) {
        flex-direction: column;
      }
    }

    .box {
      border: 1px solid #008030;
      border-radius: 5px;
      display: flex;
      align-items: center;

      span {
        margin-left: 10px;
        width: 200px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
    }

    .box2 {
      display: flex;
      align-items: center;
      gap: 10px;

      input {
        background: transparent;
        padding: 5px 10px;
        border: 1px solid #404040;
        border-radius: 5px;

        &:focus {
          outline: none;
          border-color: #008030;
        }

        &::placeholder {
          color: #a3a3a3;
        }

        &:disabled {
          cursor: not-allowed;
        }
      }
    }

    input[type="checkbox"] {
      width: 20px;
      height: 20px;
      accent-color: #008030;
      cursor: pointer;
    }

    .deleteButton {
      display: flex;
      align-items: center;
      padding: 5px;
      cursor: pointer;
    }

    .checked {
      text-decoration: line-through;
      font-style: italic;
      background-color: #d4d4d4;
      cursor: not-allowed;
    }
  }

  .containerPrice {
    position: fixed;
    background-color: #008030;
    bottom: 0;
    width: 100%;
    padding: 10px 0;

    display: flex;
    flex-direction: column;
    align-items: flex-end;
    color: #ffffff;
    gap: 5px;

    strong,
    span {
      margin: 0 20px;
    }
  }

  .addButton {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    margin-bottom: 50px;

    z-index: 5;

    .infoButton {
      width: fit-content;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;

      background-color: #ffffff;
      border-radius: 20px;
      box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.5);

      margin: auto;

      padding: 20px 50px;

      img {
        width: 50px;
        height: 50px;
        margin-bottom: 10px;
        border-radius: 10px;
      }

      span {
        margin-bottom: 20px;
      }

      div {
        display: flex;
        gap: 10px;

        button {
          all: unset;
          cursor: pointer;

          background-color: #008030;
          color: #ffffff;
          display: flex;
          text-align: center;
          justify-content: center;
          border-radius: 5px;
          align-items: center;
          padding: 5px 10px;

          &:hover {
            background-color: #006020;
          }

          &:disabled {
            background-color: #a1a1aa;
            cursor: not-allowed;
          }
        }

        .buttonVariant {
          background-color: #ffffff;
          color: #008030;
          border: 1px solid #008030;

          &:hover {
            background-color: #008030;
            color: #ffffff;
          }

          &:disabled {
            background-color: #a1a1aa;
            cursor: not-allowed;
          }
        }
      }
    }
  }
</style>
