<script>
  export let inBuilder = true;
  export let mode = "container";
</script>

{#if inBuilder}
  <div class="repeater">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 18 18">
      <rect
        id="Canvas"
        fill="#ff13dc"
        opacity="0"
        width="18"
        height="18"
      /><path
        class="fill"
        d="M16.337,10H15.39a.6075.6075,0,0,0-.581.469A5.7235,5.7235,0,0,1,5.25,13.006l-.346-.3465L6.8815,10.682A.392.392,0,0,0,7,10.4a.4.4,0,0,0-.377-.4H1.25a.25.25,0,0,0-.25.25v5.375A.4.4,0,0,0,1.4,16a.3905.3905,0,0,0,.28-.118l1.8085-1.8085.178.1785a8.09048,8.09048,0,0,0,3.642,2.1655,7.715,7.715,0,0,0,9.4379-5.47434q.04733-.178.0861-.35816A.5.5,0,0,0,16.337,10Z"
      />
      <path
        class="fill"
        d="M16.6,2a.3905.3905,0,0,0-.28.118L14.5095,3.9265l-.178-.1765a8.09048,8.09048,0,0,0-3.642-2.1655A7.715,7.715,0,0,0,1.25269,7.06072q-.04677.17612-.08519.35428A.5.5,0,0,0,1.663,8H2.61a.6075.6075,0,0,0,.581-.469A5.7235,5.7235,0,0,1,12.75,4.994l.346.3465L11.1185,7.318A.392.392,0,0,0,11,7.6a.4.4,0,0,0,.377.4H16.75A.25.25,0,0,0,17,7.75V2.377A.4.4,0,0,0,16.6,2Z"
      />
    </svg>
  </div>
{/if}

<style>
  .repeater {
    position: absolute;
    top: 12px;
    right: 12px;
    width: 16px;
    height: 16px;
    background-color: lime;
    border-radius: 50%;
    display: grid;
    align-items: center;
    justify-items: center;
    z-index: 100;
  }
  .repeater > svg {
    width: 10px;
    height: 10px;
  }
</style>
