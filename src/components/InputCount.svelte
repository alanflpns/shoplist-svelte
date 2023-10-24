<script lang="ts">
  export let lessQuantity: () => void;
  export let moreQuantity: () => void;
  export let quantity: number;
  export let isDisabled = false;
</script>

<div class="countContainer">
  <button
    type="button"
    on:click={lessQuantity}
    disabled={isDisabled}
    class:isDisabled
  >
    -
  </button>
  <input
    type="number"
    bind:value={quantity}
    pattern="[0-9]*"
    inputmode="numeric"
    disabled={isDisabled}
    class:isDisabled
  />
  <button
    type="button"
    on:click={moreQuantity}
    disabled={isDisabled}
    class:isDisabled
  >
    +
  </button>
</div>

<style scoped lang="scss">
  .countContainer {
    display: flex;

    button {
      all: unset;
      cursor: pointer;

      border: 2px solid #008030;
      color: #008030;
      width: 20px;
      display: flex;
      text-align: center;
      justify-content: center;
      border-radius: 5px;
      align-items: center;
      text-decoration: none !important;
      font-style: normal !important;

      &:disabled {
        cursor: not-allowed;
      }
    }

    input {
      width: fit-content;
      padding: 5px 10px;
      margin: 0 5px;
      max-width: 30px;

      background: transparent;
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

    input::-webkit-outer-spin-button,
    input::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }

    /* Firefox */
    input[type="number"] {
      -moz-appearance: textfield;
    }

    .disabled {
      text-decoration: line-through;
      font-style: italic;
      background-color: #d4d4d4;
      cursor: not-allowed;
    }
  }
</style>
