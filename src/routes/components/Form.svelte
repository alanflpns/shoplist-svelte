<script lang="ts">
  import Input from "../../components/Input.svelte";
  import InputCount from "../../components/InputCount.svelte";
  import { createEventDispatcher } from "svelte";
  import LocalStorageAdapter from "../../infra/LocalStorageAdapter";

  const dispatch = createEventDispatcher();

  let nameProduct = "";
  let quantity = 1;
  export let productsList: any = [];

  $: isDisabled = nameProduct.trim().length < 1;

  function handleMoreQuantity() {
    quantity += 1;
  }

  function handleLessQuantity() {
    if (quantity > 1) {
      quantity -= 1;
    }
  }

  function submitForm(event: any) {
    event.preventDefault();

    const product = {
      name: nameProduct,
      quantity,
      isChecked: false,
      price: "R$ 0,00",
    };

    if (productsList.some((product: any) => product.name === nameProduct)) {
      const productIndex = productsList.findIndex(
        (product: any) => product.name === nameProduct
      );
      const newProductsList = [...productsList];
      newProductsList[productIndex].quantity += quantity;
      dispatch("updateProductsList", newProductsList);
    } else {
      dispatch("updateProductsList", [...productsList, product]);
    }

    nameProduct = "";
    quantity = 1;

    const storage = new LocalStorageAdapter();
    storage.set("@ShopList/products", [...productsList, product]);
  }
</script>

<section>
  <form on:submit={submitForm}>
    <div class="field">
      <label for="nameProduct">Produto</label>
      <Input bind:value={nameProduct} autofocus />
    </div>
    <div class="field">
      <label for="quantity">Quantidade</label>

      <InputCount
        bind:quantity
        moreQuantity={handleMoreQuantity}
        lessQuantity={handleLessQuantity}
      />
    </div>
    <button class="buttonForm" disabled={isDisabled}> Adicionar </button>
  </form>
</section>

<style scoped lang="scss">
  form {
    width: 100%;
    display: flex;
    justify-content: center;
    gap: 10px;
    flex-wrap: wrap;

    label {
      font-size: 14px;
      font-weight: 500;
      color: #404040;
    }

    .field {
      display: flex;
      flex-direction: column;
    }

    .buttonForm {
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
  }
</style>
