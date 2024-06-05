<script>
  import { getContext, onDestroy, onMount, setContext } from "svelte";
  import "@spectrum-css/opacitycheckerboard/dist/index-vars.css";
  import RepeaterPreview from "./RepeaterPreview.svelte";
  import TabControl from "./TabControl.svelte";
  import Grabber from "./Grabber.svelte";
  import fsm from "svelte-fsm";
  import { writable } from "svelte/store";
  import CellSkeleton from "./CellSkeleton.svelte";

  const {
    styleable,
    builderStore,
    Provider,
    ContextScopes,
    componentStore,
    memo,
  } = getContext("sdk");

  const component = getContext("component");

  // Get Wrapper Super Container Stores
  const parentState = getContext("superContainer");
  const parentGridStore = getContext("superContainerParams");

  export let dataprovider;
  export let sourceArray;
  export let bound = false;
  export let hideIfEmpty = false;
  export let emptyMessage = "No Records Found";

  export let flex;
  export let flexFactor = 1;
  export let mode = "container";
  export let childMode = "containerItem";
  export let direction;
  export let hAlign;
  export let vAlign;
  export let gap;
  export let wrap;

  export let tabsSize;
  export let tabsAlignment;
  export let tabsQuiet;
  export let tabsEmphasized;
  export let activeTab = 0;
  export let tabsIconsOnly = false;
  export let theme;

  export let gridColumns = 3;
  export let gridRows = 3;
  export let title, icon, color;

  export let labelPos;
  export let labelWidth = "6rem";
  export let disabled;

  export let borderTop;
  export let borderRight;
  export let borderBottom;
  export let borderLeft;

  // Grid Child Item Options
  export let colSpan = 1;
  export let rowSpan = 1;

  export let skeleton = false;

  // Events
  export let onTabChange;

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
  let scope = ContextScopes.Local;

  const props = memo($$props);

  $: props.set($$props);

  // The array of slots to be rendered in array repeater mode
  let slots;

  // The State machine that handles the parent role of the super container
  const state = fsm(mode, {
    "*": {
      registerContainer(
        componentID,
        id,
        state,
        title,
        icon,
        color,
        reqcolSpan,
        reqrowSpan
      ) {
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
            rowSpan: reqrowSpan,
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
        this.refresh($builderStore.inBuilder);
        childState.refresh();
        $component.styles = {
          ...$component.styles,
          normal: {
            ...$component.styles.normal,
            ...cssVariables,
            ...childCssVariables,
            "border-top-width": borderTop
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "border-right-width": borderRight
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "border-bottom-width": borderBottom
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "border-left-width": borderLeft
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "--random-color": "#" + randomColor,
            "--spectrum-opacity-checkerboard-square-dark":
              "var(--spectrum-global-color-gray-200)",
            "--spectrum-opacity-checkerboard-square-light": bound
              ? "var(--random-color)"
              : "var(--spectrum-global-color-gray-75)",
            "--spectrum-opacity-checkerboard-square-size": "8px",
            "--spectrum-opacity-checkerboard-position": "left top",
          },
        };
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
          gap: gap + "rem",
          "--container-flex-mode":
            (direction == "row" && hAlign == "stretch") ||
            (direction == "column" && vAlign == "stretch")
              ? "1"
              : null,
        };
      },
    },
    grid: {
      _enter() {
        this.refresh();
      },
      refresh() {
        cssVariables = {
          display: "grid",
          "justify-items": hAlign,
          "align-items": vAlign,
          "--grid-columns": gridColumns,
          "--grid-rows": gridRows,
          "--grid-column-gap": gap + "rem",
          "--grid-row-gap": gap + "rem",
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
        if (selectedTab) this.selectTab(selectedTab);
        else {
          if (containers.length > 0) this.selectTab(containers[0].id);
        }
        cssVariables = {
          "flex-direction": direction == "row" ? "column" : "row",
        };
      },
      selectTab(tabId) {
        if (tabId == selectedTab) return;

        containers.forEach(({ id, state, title }) => {
          if (tabId == id) {
            state.show();
            onTabChange?.({ tabTitle: title });
            selectedTab = id;
          } else {
            state.hide();
          }
        });
      },
    },
    fieldgroup: {
      _enter() {
        this.refresh();
      },
      refresh() {
        cssVariables = {
          display: "grid",
          "justify-items": "stretch",
          "align-items": vAlign,
          "--grid-columns": gridColumns * 6,
          "--grid-rows": gridRows,
          "--grid-column-gap":
            labelPos == "left" ? 0.85 * gap + "rem" : 0.75 * gap + "rem",
          "--grid-row-gap":
            labelPos == "left" ? 0.75 * gap + "rem" : 0.5 * gap + "rem",
        };
      },
    },
  });

  // The State machine that handles the child role of the super container if nested
  const childState = fsm(childMode ?? "containerItem", {
    "*": {
      synch(parentState) {
        if (parentState == "grid") {
          return "gridItem";
        }
        if (parentState == "tabs") {
          return "tabItem";
        }
        if (parentState == "accordion") return "accordionItem";
        if (parentState == "splitview") return "splitviewItem";
        if (parentState == "fieldgroup") return "fieldgroupItem";
        if (!parentState || parentState == "container") {
          return "containerItem";
        }
        $component.styles = {
          ...$component.styles,
          normal: {
            ...$component.styles.normal,
            ...cssVariables,
            ...childCssVariables,
            "border-top-width": borderTop
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "border-right-width": borderRight
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "border-bottom-width": borderBottom
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "border-left-width": borderLeft
              ? $component.styles.normal["border-width"] ?? 0
              : 0,
            "--random-color": "#" + randomColor,
            "--spectrum-opacity-checkerboard-square-dark":
              "var(--spectrum-global-color-gray-200)",
            "--spectrum-opacity-checkerboard-square-light": bound
              ? "var(--random-color)"
              : "var(--spectrum-global-color-gray-75)",
            "--spectrum-opacity-checkerboard-square-size": "8px",
            "--spectrum-opacity-checkerboard-position": "left top",
          },
        };
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
          flex: flex == "grow" ? flexFactor + " 1 auto" : "0 1 auto",
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
          flex: flexFactor + " 1 auto",
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
        if (parentGridStore) {
          childCssVariables = {
            "grid-column":
              "span " + Math.min(colSpan, $parentGridStore?.gridColumns),
            "grid-row": "span " + Math.min(rowSpan, $parentGridStore?.gridRows),
            override: "hidden",
          };
        } else {
          childCssVariables = {
            "grid-column": "span " + colSpan * 6,
            "grid-row": "span " + rowSpan,
          };
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
    activeTabItem: {
      _enter() {
        this.refresh();
      },
      refresh() {
        childCssVariables = {
          flex: "1 0 auto",
        };
      },
    },
    fieldgroupItem: {
      _enter() {
        this.refresh();
      },
      refresh() {
        childCssVariables = {
          fieldgroupitem: "yes",
          "grid-column": "span " + colSpan * 6,
          "grid-row": "span " + rowSpan,
        };
      },
    },
  });

  let gridStore = new writable({});
  $: $gridStore = { gridColumns, gridRows };
  $: coords = new Array(gridColumns * gridRows);

  $: randomColor =
    $builderStore.inBuilder && bound
      ? "32CD3230"
      : Math.floor(Math.random() * 16777215).toString(16) + "10";

  $: nested = component
    ? $component.ancestors[$component.ancestors.length - 2] ==
      "plugin/bb-component-SuperContainer"
    : false;

  $: state.synchProperties($$props);

  $: if (bound == "array") slots = safeParse(sourceArray);
  $: childState.synch($parentState);
  $: parentState?.updateContainer(
    id,
    title,
    icon,
    color,
    Math.min(colSpan, $parentGridStore?.gridColumns),
    Math.min(rowSpan, $parentGridStore?.gridRows)
  );

  $: {
    if (
      $builderStore.inBuilder &&
      parentState &&
      $componentStore.selectedComponentPath?.includes($component.id)
    ) {
      parentState.selectChild($component.id);
      if (childMode != $parentState + "Item")
        builderStore.actions.updateProp("childMode", $parentState + "Item");
    } else if (
      $builderStore.inBuilder &&
      !parentState &&
      childMode != "containerItem" &&
      $componentStore.selectedComponentPath?.includes($component.id)
    ) {
      builderStore.actions.updateProp("childMode", "containerItem");
    }
  }

  $: if (mode == "grid") setContext("superContainerParams", gridStore);

  $: if (mode == "fieldgroup") {
    setContext("field-group", labelPos);
    setContext("field-group-label-width", labelWidth);
    setContext("field-group-disabled", disabled);
  }

  function safeParse(str) {
    let parsed = [];

    try {
      parsed = JSON.parse(str);
    } catch (error) {
      console.debug(error);
    }
    return parsed;
  }

  onMount(() => {
    if (mode == "tabs" && containers.length > 0) {
      if (Number(activeTab) >= 0 && Number(activeTab) < containers.length)
        state.selectTab(containers[Number(activeTab)].id);
    }
    if (parentState && nested) {
      parentState.registerContainer(
        componentID,
        id,
        state,
        title,
        icon,
        color,
        colSpan,
        rowSpan
      );
    }
  });

  onDestroy(() => {
    if (parentState && nested) {
      parentState.unregisterContainer(id);
    }
  });

  $: state.synchProperties($props);

  setContext("superContainer", state);
</script>

<svelte:window
  on:mouseup={state.stopResizing}
  on:mousemove={(e) => (resizing ? state.resize(e) : null)}
/>

{#key disabled}
  {#key mode}
    {#if $childState != "hidden" && $childState != "tabItem"}
      <div
        bind:this={container}
        class:super-container={$state == "container"}
        class:accordion={$state == "accordion"}
        class:super-grid={$state == "grid"}
        class:tabs={$state == "tabs"}
        class:splitview={$state == "splitview"}
        class:super-fieldgroup={$state == "fieldgroup"}
        class:super-container-item={$childState == "containerItem"}
        class:accordion-item={$childState == "accordionItem"}
        class:tab-item={$childState == "tabItem" || $childState == "hidden"}
        class:splitview-item={$childState == "splitviewItem"}
        class:super-fieldgroup-item={$childState == "fieldgroupItem"}
        class:nested={$builderStore.inBuilder && nested}
        class:spectrum-OpacityCheckerboard={$builderStore.inBuilder &&
          $component.empty}
        use:styleable={$component.styles}
      >
        {#if skeleton}
          <CellSkeleton>Loading ...</CellSkeleton>
        {:else}
          {#if mode == "grid" && $builderStore.inBuilder}
            <div class="underlay">
              {#each coords as _, idx}
                <div class="placeholder spectrum-OpacityCheckerboard">
                  {idx}
                </div>
              {/each}
            </div>
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
            {#if dataprovider?.rows?.length}
              {#each dataprovider.rows as row}
                <Provider data={row} {scope}>
                  <slot />
                </Provider>
              {/each}
            {:else if $builderStore.inBuilder}
              <Provider data={{}} {scope}>
                <slot />
              </Provider>
            {/if}
            <!-- In Repeater Mode with Array -->
          {:else if bound == "array"}
            {#if slots?.length}
              {#each slots as row, idx}
                <Provider
                  data={{ index: idx, value: row }}
                  scope={ContextScopes.Local}
                >
                  <slot />
                </Provider>
              {/each}
            {:else if $builderStore.inBuilder}
              <Provider
                data={{ index: -1, value: {} }}
                scope={ContextScopes.Local}
              >
                <slot />
              </Provider>
            {/if}
            <!-- In unbound mode -->
          {:else if mode == "grid"}
            <slot />
          {:else}
            {#key labelPos + labelWidth}
              <slot />
            {/key}
          {/if}

          {#if grabberPosition}
            <Grabber {grabberPosition} {resizing} {state} />
          {/if}
        {/if}
      </div>
    {/if}
  {/key}
{/key}

<style>
  :global(.super-fieldgroup > .component > *) {
    grid-column: span 6;
  }

  :global(.super-grid > .component > *) {
    overflow: hidden;
  }
  .super-container {
    display: flex;
    position: relative;
    overflow: hidden;
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

  .underlay {
    position: absolute;
    z-index: -1;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: grid;
    grid-template-columns: repeat(var(--grid-columns), 1fr);
    grid-template-rows: repeat(var(--grid-rows), 1fr);
    column-gap: var(--grid-column-gap);
    row-gap: var(--grid-row-gap);
    background-color: var(--spectrum-global-color-gray-200);
  }

  .placeholder {
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--spectrum-global-color-gray-500);
    border: 1px solid var(--spectrum-global-color-gray-400);
  }

  .super-fieldgroup {
    display: grid;
    position: relative;
    grid-template-columns: repeat(var(--grid-columns), 1fr);
    column-gap: var(--grid-column-gap);
    row-gap: var(--grid-row-gap);
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

  .spectrum-OpacityCheckerboard {
    block-size: unset;
    inline-size: unset;
  }
</style>
