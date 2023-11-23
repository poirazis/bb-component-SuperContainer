<script>
  import { getContext, onDestroy, onMount, setContext } from "svelte";
  import "@spectrum-css/opacitycheckerboard/dist/index-vars.css";
  import RepeaterPreview from "./RepeaterPreview.svelte";
  import TabControl from "./TabControl.svelte";
  import Grabber from "./Grabber.svelte";

  import fsm from "svelte-fsm";
  import { v4 as uuidv4 } from "uuid";

  const { styleable, builderStore, Provider, componentStore } =
    getContext("sdk");
  const component = getContext("component");
  const parentState = getContext("superLayoutManager");

  export let dataprovider;
  export let bound;
  export let flex;
  export let flexFactor = 1;
  export let mode = "container";
  export let direction;
  export let hAlign;
  export let vAlign;
  export let gap;
  export let wrap;

  export let tabsSize;
  export let tabsAlignment;
  export let tabsQuiet;
  export let activeTab = 0
  export let theme;

  export let gridColumns = 3;
  export let gridRows = 3;

  export let title, icon, color;

  export let onClick;

  // Grid Child Item Options
  export let colSpan = 1
  export let rowSpan = 1

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
  let id = uuidv4();
  let childCssVariables = {};
  let cssVariables = {};

  const childState = fsm( "containerItem" , {
    "*": {
      synch(parentState) {
        if (parentState == "grid") { return "gridItem"; }
        if (parentState == "tabs") { return "tabItem"; }
        if (!parentState || parentState == "container") { return "containerItem";}
        if (parentState == "accordion") return "accordionItem";
        if (parentState == "splitview") return "splitviewItem";
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
          flex: flexFactor + " 0 auto",
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
        childCssVariables = {
          width: "100%",
          height: "100%",
          "grid-column" : "span " + colSpan,
          "grid-row" : "span " + rowSpan
        };
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

  const state = fsm(mode, {
    "*": {
      registerContainer(componentID, id, state, title, icon, color) {
        containers = [
          ...containers,
          {
            componentID: componentID,
            id: id,
            state: state,
            title: title,
            icon: icon,
            color: color,
          },
        ];
      },
      updateContainer(id, title, icon, color) {
        let index = containers.findIndex((e) => e.id == id);
        if (index > -1) {
          containers[index].title = title;
          containers[index].icon = icon;
          containers[index].color = color;
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
        childState.refresh();
        this.refresh();
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
          display: "flex",
          "flex-direction": direction,
          "flex-wrap": wrap ? "wrap" : "nowrap",
          "justify-content": direction == "row" ? hAlign : vAlign,
          "align-items": direction == "row" ? vAlign : hAlign,
          "align-content": wrap ? (direction == "row" ? vAlign : hAlign) : null,
          gap: gap,
        };
      },
    },
    grid: {
      _enter() {
        this.refresh();
      },
      refresh() {
        cssVariables = {
          "justify-items": hAlign,
          "align-items": vAlign,
          "--grid-columns": gridColumns,
          "--grid-rows": gridRows,
          "--grid-gap": gap,
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

  $: randomColor = bound
    ? "32CD3230"
    : Math.floor(Math.random() * 16777215).toString(16) + "50";
  $: nested = component
    ? $component.ancestors[$component.ancestors.length - 2] ==
      "plugin/bb-component-SuperContainer"
    : false;

  $: state.synchProperties($$props);
  $: if (parentState && nested) childState.synch($parentState);
  $: parentState?.updateContainer(id, title, icon, color);

  $: {
    if (
      $builderStore.inBuilder &&
      parentState &&
      $componentStore.selectedComponentPath?.includes($component.id)
    ) {
      parentState.selectChild($component.id);
      childState.synch($parentState)
      builderStore.actions.updateProp("childMode", $parentState+"Item")
    }
  }

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      ...childCssVariables,
      ...cssVariables,
      "--random-color": "#" + randomColor,
      overflow: "auto",
      "--spectrum-opacity-checkerboard-square-dark":
        "var(--spectrum-global-color-gray-50)",
      "--spectrum-opacity-checkerboard-square-light": bound
        ? "var(--random-color)"
        : "var(--spectrum-global-color-gray-75)",
      "--spectrum-opacity-checkerboard-square-size": "8px",
      "--spectrum-opacity-checkerboard-position": "left top",
    },
  };

  onMount(() => {
    if ( mode == "tabs" && containers.length > 0 ) 
    {
      if ( Number(activeTab) >= 0 && Number(activeTab) < containers.length ) 
        state.selectTab(containers[Number(activeTab)].id)
      else
        state.selectTab(containers[0].id)
    }
    if (parentState && nested) {
      parentState.registerContainer(componentID, id, state, title, icon, color);
    }
  });

  onDestroy(() => {
    if (parentState && nested) {
      parentState.unregisterContainer(id);
    }
  });

  setContext("superLayoutManager", state);
</script>

<svelte:window
  on:mouseup={state.stopResizing}
  on:mousemove={(e) => (resizing ? state.resize(e) : null)}
/>

{#if $childState != "hidden" && $childState != "tabItem" }
  <div
    bind:this={container}
    class:container-item={$childState == "containerItem"}
    class:accordion-item={$childState == "accordionItem"}
    class:grid-item={$childState == "gridItem"}
    class:tab-item={$childState == "tabItem" || $childState == "hidden"}
    class:splitview-item={$childState == "splitviewItem"}
    class:container={$state == "container"}
    class:accordion={$state == "accordion"}
    class:grid={$state == "grid"}
    class:tabs={$state == "tabs"}
    class:splitview={$state == "splitview"}
    class:nested={$builderStore.inBuilder && nested}
    class:spectrum-OpacityCheckerboard={$builderStore.inBuilder}
    use:styleable={$component.styles}
  >
    {#if mode == "tabs" && containers.length > 0}
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
      />
    {/if}

    {#if bound && dataprovider}
      <RepeaterPreview inBuilder={$builderStore.inBuilder} {mode} />
      {#each dataprovider.rows as row}
        <Provider data={row}>
          <slot />
        </Provider>
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
  .container {
    position: relative;
  }
  .container-item {
    flex: var(--flex-factor);
  }
  .grid {
    display: grid;
    position: relative;
    grid-template-columns: repeat(var(--grid-columns), 1fr);
    grid-template-rows: repeat(var(--grid-rows), 1fr);
    grid-column-gap: var(--grid-gap);
    grid-row-gap: var(--grid-gap);
  }
  .grid-item {
    display: grid;
    grid-column-start: auto;
    grid-column-end: auto;
    grid-row-start: auto;
    grid-row-end: auto;
  }
  .accordion {
    position: relative;
  }

  .accordion-item {
    border: 4px solid lime;
  }
  .tabs {
    display: flex;
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
  .spectrum-OpacityCheckerboard {
    block-size: unset;
    inline-size: unset;
  }
</style>