<script>
  import "@spectrum-css/splitview"
  import Section from "./Section.svelte";

  export let direction
  export let resizable
  export let collapsible


</script>

<div class="spectrum-SplitView spectrum-SplitView--horizontal" style="height: 200px">
  <div class="spectrum-SplitView-pane" style="width: 304px">Left</div>
  <div class="spectrum-SplitView-splitter is-draggable">
    <div class="spectrum-SplitView-gripper"></div>
  </div>
  <div class="spectrum-SplitView-pane" style="flex: 1">Right</div>
</div>
