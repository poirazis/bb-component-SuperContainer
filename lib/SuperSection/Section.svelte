<script>
  export let width = "50%"
</script>

<div class="superSection">
  This is a Section
</div>

<style>
  .superSection {
    flex: auto;
    position: relative;
    border: 1px solid var(--spectrum-global-color-gray-400);
  }

  .superSection::after {
    position: absolute;
    content: "";
    width: 20px;
    height: 100%;
    right: 0;
    top: 0;
    background-color: var(--spectrum-global-color-gray-200);
    z-index: 1;
  }
  .superSection:hover::after {
    background-color: var(--spectrum-global-color-gray-400);
  }
</style>