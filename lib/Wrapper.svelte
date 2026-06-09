<script>
  import Component from "../src/Component.svelte";

  let props = $props();
</script>

<svelte:boundary
  onerror={(error) => {
    console.error("[SuperContainer]", error);
  }}
>
  <Component {...props}>
    {@render props.children?.()}
  </Component>

  {#snippet failed(error)}
    <div class="plugin-error" role="alert">
      <span class="plugin-error-title">Container failed to render</span>
      {#if error instanceof Error && error.message}
        <span class="plugin-error-detail">{error.message}</span>
      {/if}
    </div>
  {/snippet}
</svelte:boundary>

<style>
  .plugin-error {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
    padding: 0.5rem 0.75rem;
    border: 1px solid var(--spectrum-global-color-red-400);
    border-radius: 4px;
    background: var(--spectrum-global-color-red-100);
    color: var(--spectrum-global-color-red-800);
    font-size: 13px;
    box-sizing: border-box;
    width: 100%;
  }

  .plugin-error-title {
    font-weight: 600;
  }

  .plugin-error-detail {
    font-size: 12px;
    color: var(--spectrum-global-color-red-700);
    word-break: break-word;
  }
</style>