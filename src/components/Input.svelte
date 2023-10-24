<script lang="ts">
  export let noBorder = false;
  export let width = "";
  export let autofocus = false;

  export let value = "";
</script>

<input
  {autofocus}
  placeholder="Insira o nome"
  class:noBorder
  style="width: {width};"
  bind:value
/>

<style lang="scss">
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

  .noBorder {
    border: none;
  }

  .width {
    width: 100%;
  }
</style>
