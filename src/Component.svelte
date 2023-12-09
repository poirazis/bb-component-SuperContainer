<script>
  import { getContext, onDestroy, onMount, setContext } from "svelte";
  import fsm from "svelte-fsm";

  import "@spectrum-css/opacitycheckerboard/dist/index-vars.css";
  import RepeaterPreview from "./RepeaterPreview.svelte";
  import TabControl from "./TabControl.svelte";
  import Grabber from "./Grabber.svelte";

  import { writable } from "svelte/store";

  const { styleable, builderStore, Provider, componentStore } = getContext("sdk");
  const component = getContext("component");

  const parentState = getContext("superContainer");
  const parentGridStore = getContext("superContainerParams");

  export let dataprovider
  export let sourceArray
  export let bound = false

  export let flex;
  export let flexFactor = 1;
  export let mode = "container"
  export let childMode = "containerItem"
  export let direction;
  export let hAlign;
  export let vAlign;
  export let gap;
  export let wrap;

  export let tabsSize;
  export let tabsAlignment;
  export let tabsQuiet;
  export let tabsEmphasized
  export let activeTab = 0
  export let tabsIconsOnly = false
  export let theme;

  export let gridColumns = 3;
  export let gridRows = 3;
  export let title, icon, color;

  // Grid Child Item Options
  export let colSpan
  export let rowSpan

  let containers = [];
  let container;

  let resizing;
  let startPointX;
  let startPointY;
  let width, height;
  let initialWidth;
  let initialHeight;
  let grabberPosition;
  let selectedTab = undefined;

  let componentID = $component.id;
  let id = Math.random() * 10;
  let childCssVariables = {};
  let cssVariables = {};
  let gridPreviewSlots

  // The array of slots to be rendered in array repeater mode
  let slots

  // The State machine that handles the child role of the super container if nested
  const childState = fsm( "containerItem" , {
    "*": {
      synch(parentState,inFieldGroup ) {
        if ( inFieldGroup ) {
          return "gridItem"
        } else {
          if (parentState == "grid" ) { return "gridItem"; }
          if (parentState == "tabs") { return "tabItem"; }
          if (parentState == "accordion") return "accordionItem";
          if (parentState == "splitview") return "splitviewItem";
          if (!parentState || parentState == "container") { return "containerItem";}
        }
      },
      activate() {
        return "activeTabItem";
      },
      deactivate() {
        return "hidden";
      },
    },
    hidden: {
      activate() {
        return "activeTabItem";
      },
    },
    disabled: {},
    containerItem: {
      _enter() {
        this.refresh();
      },
      refresh() {
        childCssVariables = {
          flex: flex == "grow" ? flexFactor + " 0 auto" : "0 0 auto",
          "grid-column" : "span " + colSpan,
          "grid-row" : "span " + rowSpan
        };
      },
    },
    accordionItem: {},
    splitviewItem: {
      _enter() {
        this.refresh();
      },
      refresh() {
        childCssVariables = {
          "flex": flexFactor + " 1 auto",
          "min-width": width ? width : "auto",
          "max-width": width ? width : "auto",
          "min-height": height ? height : "auto",
          "max-height": height ? height : "auto",
        };
      },
    },
    gridItem: {
      _enter() {
        this.refresh();
      },
      refresh() {
        if ( parentGridStore ) {
          childCssVariables = {
            "grid-column" : "span " + Math.min( colSpan, $parentGridStore?.gridColumns ),
            "grid-row" : "span " + Math.min( rowSpan, $parentGridStore?.gridRows),
          } 
        } else {
          childCssVariables = {
            "grid-column" : "span " + (colSpan * 6),
            "grid-row" : "span " + rowSpan,
          } 
        }
      },
    },
    tabItem: {
      _enter() {
        this.refresh();
        },
      },
      refresh() {
        childCssVariables = {
          flex: "1 0 auto",
        };
      },
    activeTabItem : {
      _enter() {
        this.refresh();
      },
      refresh() {
        childCssVariables = {
          flex: "1 0 auto",
        }
      }
    }
  });

  // The State machine that handles the parent role of the super container 
  const state = fsm("container", {
    "*": {
      registerContainer(componentID, id, state, title, icon, color, reqcolSpan, reqrowSpan) {
        containers = [
          ...containers,
          {
            componentID,
            id,
            state,
            title,
            icon,
            color,
            colSpan: reqcolSpan,
            rowSpan: reqrowSpan
          },
        ];
      },
      updateContainer(id, title, icon, color, reqcolSpan = 1, reqrowSpan = 1) {
        let index = containers.findIndex((e) => e.id == id);
        if (index > -1) {
          containers[index].title = title;
          containers[index].icon = icon;
          containers[index].color = color;
          containers[index].colSpan = reqcolSpan;
          containers[index].rowSpan = reqrowSpan;
        }
        containers = containers;
      },
      unregisterContainer(id) {
        let index = containers.findIndex((e) => e.id == id);
        if (index > -1) {
          containers.splice(index, 1);
          containers = containers;
          if (mode == "tabs" && containers.length > 0)
            selectedTab = containers[0].id;
        }
      },
      setMode(mode) {
        return mode;
      },
      refresh() {},
      selectChild(compID) {
        if (mode == "tabs") {
          let pos = containers.findIndex((v) => v.componentID == compID);
          if (pos > -1) this.selectTab(containers[pos].id);
        }
      },
      setResizing(pos) {
        grabberPosition = pos;
      },
      startResizing(e) {
        resizing = true;
        startPointX = e.clientX;
        startPointY = e.clientY;
        initialWidth = container.clientWidth;
        initialHeight = container.clientHeight;
      },
      resize(e) {
        if (grabberPosition == "right") {
          width = initialWidth + (e.clientX - startPointX);
        } else {
          height = initialHeight + (e.clientY - startPointY);
        }

        childState.refresh();
      },
      stopResizing(e) {
        resizing = false;
      },
      hide() {
        childState.deactivate();
      },
      show() {
        childState.activate();
      },
      synchProperties() {
        childState.refresh( );
        this.refresh( $builderStore.inBuilder  );
        return mode;
      },
    },
    disabled: {},
    accordion: {},
    container: {
      _enter() {
        this.refresh();
      },
      refresh() {
        cssVariables = {
          "flex-direction": direction,
          "flex-wrap": wrap ? "wrap" : "nowrap",
          "justify-content": direction == "row" ? hAlign : vAlign,
          "align-items": direction == "row" ? vAlign : hAlign,
          "align-content": wrap ? (direction == "row" ? vAlign : hAlign) : null,
          "gap": gap,
          "--container-flex-mode" : ( direction == "row" && hAlign == "stretch" ) || ( direction == "column" && vAlign == "stretch" ) ? "1" : null
        };
      },
    },
    grid: {
      _enter() {
        this.refresh( $builderStore.inBuilder );
      },
      refresh( inBuilder ) {
        let repeaterUsedSlots = 0
        let nonSuperComponents = 0
        let totalSlots = gridColumns * gridRows
        let childUsedSlots = containers?.reduce( ( p , c , idx ) => { return p + ( c.colSpan * c.rowSpan ) }, 0 ) 

        if ( inBuilder ) {
          if ( bound == "dataprovider" && dataprovider ) {
            nonSuperComponents = $component.children - ( containers.length / dataprovider?.rows.length );
            repeaterUsedSlots = $component.children ? dataprovider?.rows.length * nonSuperComponents : dataprovider?.rows.length
          } else if ( bound == "array" && slots?.length ) {
            nonSuperComponents = $component.children - ( containers.length / slots.length );
            repeaterUsedSlots = $component.children ? slots.length * nonSuperComponents : slots.length
          } else {
            nonSuperComponents = $component.children - containers?.length
          }
          
          
          let neededPreviewSlots = totalSlots - repeaterUsedSlots - childUsedSlots - nonSuperComponents
          gridPreviewSlots = neededPreviewSlots > 0 ? new Array( neededPreviewSlots ) : []
        }

        cssVariables = {
          "display": "grid",
          "justify-items": hAlign,
          "align-items": vAlign,
          "--grid-columns": gridColumns,
          "--grid-rows": gridRows,
          "--grid-column-gap": gap,
          "--grid-row-gap": gap,
        };
      },
    },
    splitview: {
      _enter() {
        this.refresh();
      },
      refresh() {
        cssVariables = {
          "flex-direction": direction,
          "justify-content": "stretch",
        };
        this.setResizingGrabbers.debounce(20);
      },
      setResizingGrabbers() {
        containers.forEach(({ state }, idx) => {
          if (direction == "row" && idx < containers.length - 1)
            state.setResizing("right");
          else if (direction == "column" && idx < containers.length - 1)
            state.setResizing("bottom");
        });
      },
      _exit() {
        containers.forEach(({ state }) => {
          state.setResizing(null);
        });
      },
    },
    tabs: {
      _enter() {
        this.refresh();
      },
      _exit() {
        selectedTab == undefined;
      },
      refresh() {
        if ( selectedTab )
          this.selectTab(selectedTab)
        else {
          if (containers.length > 0) this.selectTab(containers[0].id)
        }
        cssVariables = {
          "flex-direction": direction == "row" ? "column" : "row",
        };
      },
      selectTab(tabId) {
        containers.forEach(({ id, state }) => {
          if (tabId == id) {
            state.show();
            selectedTab = id;
          } else {
            state.hide();
          }
        });
      },
    },
  });

  let gridStore = new writable({})
  $: $gridStore = { gridColumns, gridRows }
  
  $: randomColor = $builderStore.inBuilder && bound
    ? "32CD3230"
    : Math.floor(Math.random() * 16777215).toString(16) + "10";

  $: nested = component
    ? $component.ancestors[$component.ancestors.length - 2] ==
      "plugin/bb-component-SuperContainer"
    : false;  

  $: inSuperFieldGroup = component
    ? $component.ancestors[$component.ancestors.length - 2] ==
      "plugin/bb-component-SuperFieldGroup"
    : false;  

  $: if ( bound == "array" && sourceArray ) slots = safeParse(sourceArray) 

  $: childState.synch($parentState, inSuperFieldGroup);
  $: parentState?.updateContainer(id, title, icon, color, gridColumns, gridRows );

  $: {
    if (
      $builderStore.inBuilder &&
      parentState &&
      $componentStore.selectedComponentPath?.includes($component.id)
    ) {
      parentState.selectChild($component.id);
      if ( childMode != $parentState+"Item" )
        builderStore.actions.updateProp("childMode", $parentState+"Item")
    } else if (
      $builderStore.inBuilder
      && !parentState && inSuperFieldGroup &&
      $componentStore.selectedComponentPath?.includes($component.id)
    ) {
      builderStore.actions.updateProp("childMode", "gridItem")
    } else if (
      $builderStore.inBuilder
      && !parentState && childMode != "containerItem" && 
      $componentStore.selectedComponentPath?.includes($component.id)
    ) {
      builderStore.actions.updateProp("childMode", "containerItem")
    }
  }
  
  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      ...cssVariables,
      ...childCssVariables,
      "--random-color": "#" + randomColor,
      "--spectrum-opacity-checkerboard-square-dark":
        "var(--spectrum-global-color-gray-50)",
      "--spectrum-opacity-checkerboard-square-light": bound
        ? "var(--random-color)"
        : "var(--spectrum-global-color-gray-75)",
      "--spectrum-opacity-checkerboard-square-size": "8px",
      "--spectrum-opacity-checkerboard-position": "left top",
    },
  };

  $: state.synchProperties($$props);

  // Catch Erroneous states last after all reactive statememtns
  $: error = mode == "tabs" && containers?.length < 1 ? "At least one child Super Container needed to render Tabs"
           : mode == "splitview" && containers?.length < 2 ? "At least two child Super Containers needed to render a Split View"
           : bound == "dataprovider" && !dataprovider ? "Please place inside a Data Provider"
           : bound == "array" && !slots ? "Error Parsing Source Array"           
           : undefined

  function safeParse(str) {
    let parsed;

    try {
      parsed = JSON.parse(str);
    } catch (error) {
      console.debug(error)
    }
    return parsed;
  }

  onMount(() => {
    if ( mode == "tabs" && containers.length > 0 ) 
    {
      if ( Number(activeTab) >= 0 && Number(activeTab) < containers.length ) 
        state.selectTab(containers[Number(activeTab)].id)
      else
        state.selectTab(containers[0].id)
    }
    if (parentState && nested) {
      parentState.registerContainer(componentID, id, state, title, icon, color, colSpan, rowSpan);
    }
  });

  onDestroy(() => {
    if (parentState && nested) {
      parentState.unregisterContainer(id);
    }
  });

  setContext("superContainer", state);
  setContext("superContainerParams", gridStore )

</script>

<svelte:window
  on:mouseup={state.stopResizing}
  on:mousemove={(e) => (resizing ? state.resize(e) : null)}
/>

{#if $childState != "hidden" && $childState != "tabItem" }
  <div
    bind:this={container}
    class:super-container={$state == "container"}
    class:accordion={$state == "accordion"}
    class:super-grid={$state == "grid"}
    class:tabs={$state == "tabs"}
    class:splitview={$state == "splitview"}
    class:super-container-item={$childState == "containerItem"}
    class:accordion-item={$childState == "accordionItem"}
    class:tab-item={$childState == "tabItem" || $childState == "hidden"}
    class:splitview-item={$childState == "splitviewItem"}
    class:nested={$builderStore.inBuilder && nested}
    class:spectrum-OpacityCheckerboard={$builderStore.inBuilder && $component.empty}
    use:styleable={$component.styles}
  >
      {#if error && $builderStore.inBuilder}
        <p class="error"> 🔔 {error}</p>
      {/if}

      {#if mode == "tabs" && containers?.length > 0}
        <TabControl
          {containers}
          {hAlign}
          {vAlign}
          {direction}
          {selectedTab}
          {state}
          {theme}
          {tabsQuiet}
          {tabsAlignment}
          {tabsSize}
          {tabsIconsOnly}
          {tabsEmphasized}
        />
      {/if}

      <!-- In Repeater Mode with Data Provider -->
      {#if bound == "dataprovider" && dataprovider}
        <RepeaterPreview inBuilder={$builderStore.inBuilder} {mode} />
        {#each dataprovider.rows as row}
          <Provider data={row}>
            <slot />
          </Provider>
        {/each}
        {#if mode == "grid" && $builderStore.inBuilder}
          {#each gridPreviewSlots as guide, idx }
            <div class="grid-guides">
              {idx + ( gridColumns * gridRows ) - gridPreviewSlots?.length }
            </div>
          {/each}
        {/if}
      <!-- In Repeater Mode with Array -->
      {:else if bound == "array" && slots?.length}
        <RepeaterPreview inBuilder={$builderStore.inBuilder} {mode} />
        {#each slots as row, idx }
          <Provider data={ { index: idx, item:row, total: slots?.length } }>
            <slot />
          </Provider>
        {/each}
        {#if mode == "grid" && $builderStore.inBuilder}
          {#each gridPreviewSlots as guide, idx }
            <div class="grid-guides">
              {idx + ( gridColumns * gridRows ) - gridPreviewSlots?.length }
            </div>
          {/each}
        {/if}
      <!-- In unbound mode -->
      {:else if mode == "grid" && $builderStore.inBuilder && $component.empty }
        {#each gridPreviewSlots as _, idx }
          <div class="grid-guides">
            {idx + ( gridColumns * gridRows ) - gridPreviewSlots?.length }
            {#if idx == 0}
              <slot /> 
            {/if}
          </div>
        {/each}
      {:else if mode == "grid" && $builderStore.inBuilder}
        <slot />
        {#each gridPreviewSlots as _, idx }
          <div class="grid-guides">
            {idx + gridPreviewSlots?.length }
          </div>
        {/each}
      {:else}
        <slot />
      {/if}

      {#if grabberPosition}
        <Grabber {grabberPosition} {resizing} {state} />
      {/if}

  </div>
{/if}

<style>

  :global(.super-container > .component > * )  {
    flex: var(--container-flex-mode);
  }

  .super-container {
    display: flex;
    position: relative;
  }
  .super-container-item {
    flex: var(--flex-factor);
  }
  .super-grid {
    display: grid;
    position: relative;
    grid-template-columns: repeat(var(--grid-columns), 1fr);
    grid-template-rows: repeat(var(--grid-rows), 1fr);
    column-gap: var(--grid-column-gap);
    row-gap: var(--grid-row-gap);
  }

  .grid-guides {
    flex: 1 1 auto;
    border: 1px dotted var(--spectrum-global-color-gray-700);
    color: var(--spectrum-global-color-gray-500);
    display: grid;
    align-content: center;
    justify-content: center;
    align-self: stretch;
    justify-self: stretch;
  }

  .accordion {
    position: relative;
  }

  .accordion-item {
    border: 4px solid lime;
  }
  .tabs {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    justify-content: stretch;
  }

  .tab-item {
    flex: 1 1 auto;
  }

  .splitview {
    display: flex;
    align-items: stretch;
    justify-content: stretch;
  }
  .splitview-item {
    flex: 1 1 auto;
  }

  .nested {
    --spectrum-opacity-checkerboard-square-dark: var(
      --random-color
    ) !important ;
  }

  .error {
    font-size: 16px;
    font-weight: 500;
    padding: 1rem;
    width: 100%;
    border: 1px solid var(--spectrum-global-color-red-500);
    border-radius: 4px;
    background-color: var(--backgroundColoe);
  }

  .spectrum-OpacityCheckerboard {
    block-size: unset;
    inline-size: unset;
  }
</style>