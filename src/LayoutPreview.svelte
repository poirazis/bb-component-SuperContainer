<script>
  export let mode = "splitview"

</script>


{#if mode == "splitview" }
  <div class="splitview" />
{/if}

{#if mode == "tabs" }
  <div class="tabs" />
{/if}


<style>
  .splitview {
    min-height: 10rem;
    min-width: 10rem;
    flex:auto;
    display: flex;
    align-items: stretch;
    justify-items: stretch;
    font-size: 24px;
  }
  .splitview::before {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 50%;
    border: 8px solid var(--spectrum-global-color-gray-300);
    content: "Section 1";
  }
  .splitview::after {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 50%;
    border: 8px solid var(--spectrum-global-color-gray-300);
    content: "Section 2";
  }

  .tabs {
    flex:auto;
    display: flex;
    flex-direction: column;
    align-items: stretch;
    justify-items: stretch;
  }

  .tabs::before {
    width: 8rem;
    height: 3rem;
    display: flex;
    align-items: center;
    justify-content: center;
    content: "Tab 1" ;
    border: 8px solid var(--spectrum-global-color-gray-300);
  }
  .tabs::after {
    min-height: 10rem;
    flex:auto;
    display: flex;
    align-items: center;
    justify-content: center;
    content: "Contents";
    border: 8px solid var(--spectrum-global-color-gray-300);
  }

</style>