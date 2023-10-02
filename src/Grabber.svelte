<script>
  export let grabberPosition
  export let state
  export let resizing


</script>

<div 
  class="grabber" 
  class:resizing
  class:top={grabberPosition == "top"}
  class:right={grabberPosition == "right"}
  class:bottom={grabberPosition == "bottom"}
  class:left={grabberPosition == "left"}
  style:--pointer={ grabberPosition == "left" || grabberPosition == "right" ? "col-resize" : "row-resize" }
  on:mousedown={ state.startResizing }
/>

<style>
  .grabber {
    position: absolute;
    background-color: rgba(255,255,255, 0.085);
    cursor: var(--pointer);
    --grabber-thickness: 8px;
  }

  .resizing {
    background-color: rgba(255,255,255, 0.285);
  }

  .bottom {
    bottom: 0px;
    left: 0px;
    height: var(--grabber-thickness);
    width: 100%;
  }
  .right {
    top: 0px;
    right: 0px;
    width: var(--grabber-thickness);
    height: 100%;
    z-index: 2;
  }
</style>