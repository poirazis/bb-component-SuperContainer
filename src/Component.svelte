<script>
  import { getContext, onDestroy, onMount, setContext } from "svelte";
  import "@spectrum-css/opacitycheckerboard/dist/index-vars.css";
  import TabControl from "./TabControl.svelte";
  import Grabber from "./Grabber.svelte";
  import fsm from "svelte-fsm";
  import Expander from "./Expander.svelte";

  const { styleable, builderStore, memo } = getContext("sdk");

  const component = getContext("component");

  // Get Wrapper Super Container Stores
  const parentState = getContext("superContainer");
  const parentGridStore = getContext("superContainerParams");

  export let flex;
  export let flexFactor = 1;
  export let mode = "container";
  export let childMode = "containerItem";
  export let direction;
  export let hAlign = "stretch";
  export let vAlign;
  export let gap;
  export let wrap;

  export let panelTitle;
  export let collapsed;
  export let collapsible;
  export let collapseSide = "left";
  export let collapseSize;
  export let collapseTitle;
  export let collapseIcon;

  export let tabsSize = "M";
  export let tabsAlignment;
  export let quietTabs;
  export let activeTab = 0;
  export let tabsIconsOnly = false;
  export let theme = "budibase";
  export let list_title = "Settings";
  export let list_icon = "ri-settings-line";
  export let tabDisabled;

  export let gridColumns = 3;
  export let gridRows = 3;
  export let title, icon, color;

  export let labelPos;
  export let labelWidth = "6rem";
  export let disabled;

  // Grid Child Item Options
  export let colSpan = 1;
  export let rowSpan = 1;

  // Events as Parent
  export let onTabChange;
  export let onClick;
  export let onRightClick;

  export let hoverBackground;
  export let hoverBorder;
  export let hoverText;

  // Events as Child
  export let onShow;

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
  let builderCssVariables = {};

  // Memoize the props to avoid reactivity issues
  const props = memo($$props);
  $: props.set($$props);

  // The State machine that handles the parent role of the super container
  const state = fsm(mode, {
    "*": {
      registerContainer(componentID, id, state, title, icon, color, disabled) {
        containers = [
          ...containers,
          {
            componentID,
            id,
            state,
            title,
            icon,
            color,
            disabled,
          },
        ];
      },
      updateContainer(
        id,
        title,
        icon,
        color,
        tabDisabled,
        reqcolSpan = 1,
        reqrowSpan = 1
      ) {
        let index = containers.findIndex((e) => e.id == id);
        if (index > -1) {
          containers[index].title = title;
          containers[index].icon = icon;
          containers[index].color = color;
          containers[index].disabled = tabDisabled;
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

        childState?.refresh?.();
      },
      stopResizing(e) {
        resizing = false;
      },
      hide() {
        childState.deactivate();
      },
      show() {
        onShow?.();
        childState.activate();
      },
      synchProperties() {
        this.refresh();
        childState?.refresh?.();
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

        if (collapsible) {
          cssVariables["padding-top"] =
            collapseSide != "top" ? "2.4rem" : "unset";
          cssVariables["padding-bottom"] =
            collapseSide == "top" ? "2.4rem" : "unset";
          if (collapsed) {
            if (collapseSide == "left" || collapseSide == "right") {
              cssVariables["min-width"] =
                collapseSize ?? "3rem " + " !important";
              cssVariables["max-width"] =
                collapseSize ?? "3rem " + " !important";
            } else
              cssVariables["height"] = collapseSize ?? "3rem " + " !important";
          } else {
            cssVariables["min-width"] = "unset";
            cssVariables["max-width"] = "unset";
            cssVariables["height"] = "unset";
          }
        }

        builderCssVariables = {
          "--spectrum-opacity-checkerboard-square-dark":
            "var(--spectrum-global-color-gray-75)",
          "--spectrum-opacity-checkerboard-square-light":
            "var(--spectrum-global-color-gray-100)",
          "--spectrum-opacity-checkerboard-square-size": "8px",
          "--spectrum-opacity-checkerboard-position": "left top",
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

        builderCssVariables = {};
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
          "flex-direction":
            direction == "column" || theme == "list" ? "row" : "column",
          ...(theme == "list" && {
            border: "1px solid var(--spectrum-global-color-gray-300)",
          }),
        };
      },
      selectTab(tabId) {
        if (tabId == selectedTab) return;
        else {
          selectedTab = tabId;
          onTabChange?.({ tabTitle: title });
        }
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
          "--grid-row-gap": "0.25rem",
        };
      },
    },
  });

  // The State machine that handles the child role of the super container if nested
  const childState = fsm(childMode ?? "containerItem", {
    "*": {
      synch(parentState) {
        if (!parentState || parentState == "container") {
          return "containerItem";
        } else return parentState + "Item";
      },
    },
    disabled: {},
    containerItem: {
      _enter() {
        this.refresh();
      },
      refresh() {
        childCssVariables = {
          flex: flex == "grow" ? flexFactor + " 1 auto" : "0 0 auto",
        };
      },
    },
    panelItem: {
      _enter() {
        this.refresh();
      },
      refresh() {
        childCssVariables = {
          flex: flex == "grow" ? flexFactor + " 1 auto" : "0 0 auto",
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

          builderCssVariables = {};
        } else {
          childCssVariables = {
            "grid-column": "span " + colSpan * 6,
            "grid-row": "span " + rowSpan,
          };
        }
      },
    },
    tabsItem: {
      _enter() {
        this.refresh();
      },
      activate() {
        return "activeTabsItem";
      },
      refresh() {
        childCssVariables = {
          flex: "auto",
        };
      },
    },
    activeTabsItem: {
      _enter() {
        this.refresh();
      },
      deactivate() {
        return "tabsItem";
      },
      refresh() {
        childCssVariables = {
          flex: "auto",
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

  $: coords = new Array(gridColumns * gridRows);
  $: childState?.synch?.($parentState);

  // Update on property changes
  $: state.synchProperties($props);

  $: if ($childState == "tabsItem" && $parentGridStore?.selectedTab == id)
    childState.activate();
  else if (
    $childState == "activeTabsItem" &&
    $parentGridStore?.selectedTab != id
  )
    childState.deactivate();

  // If a Child , keep in sync with parent
  $: parentState?.updateContainer(id, title, icon, color, tabDisabled);

  // Inside Builder specigic code
  $: inBuilder = $builderStore.inBuilder;
  $: selected = $component.selected || $component.inSelectedPath;

  $: {
    if (inBuilder && selected && parentState) {
      parentState.selectChild($component.id);
      if (childMode != $parentState + "Item") {
        builderStore.actions.updateProp("childMode", $parentState + "Item");
        childMode = $parentState + "Item";
      }
    } else if (
      inBuilder &&
      selected &&
      !parentState &&
      childMode != "containerItem"
    ) {
      builderStore.actions.updateProp("childMode", "containerItem");
    }
  }

  $: if (mode == "fieldgroup") {
    setContext("field-group", labelPos);
    setContext("field-group-columns", gridColumns);
    setContext("field-group-label-width", labelWidth);
    setContext("field-group-disabled", disabled);
  } else {
    setContext("field-group", null);
    setContext("field-group-columns", null);
    setContext("field-group-label-width", null);
    setContext("field-group-disabled", null);
  }

  onMount(() => {
    if (mode == "tabs" && containers.length > 0) {
      if (Number(activeTab) >= 0 && Number(activeTab) < containers.length)
        state.selectTab(containers[Number(activeTab)].id);
    }
    parentState?.registerContainer(
      componentID,
      id,
      state,
      title,
      icon,
      color,
      tabDisabled
    );
  });

  onDestroy(() => {
    parentState?.unregisterContainer(id);
  });

  const params = memo({});
  $: params.set({
    gridColumns,
    gridRows,
    selectedTab,
    theme,
  });

  const handleCollapse = () => {
    collapsed = !collapsed;
    state.synchProperties();
  };

  // Expose State to Children
  setContext("superContainer", state);
  setContext("superContainerParams", params);

  // Append Compnent Styles
  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      ...cssVariables,
      ...childCssVariables,
      ...builderCssVariables,
      "--container-hover-background": hoverBackground,
      "--container-hover-border": hoverBorder,
      "--container-hover-color": hoverText,
    },
  };
</script>

<svelte:window
  on:mouseup={resizing ? state.stopResizing : () => {}}
  on:mousemove={(e) => (resizing ? state.resize(e) : null)}
/>
{#key mode}
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  {#if $childState != "tabsItem"}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div
      bind:this={container}
      class:collapsed
      class:hoverable={hoverBackground || hoverBorder || hoverText}
      class:collapsible
      class:clickable={onClick}
      class:super-container={$state == "container"}
      class:super-grid={$state == "grid"}
      class:tabs={$state == "tabs"}
      class:splitview={$state == "splitview"}
      class:super-fieldgroup={$state == "fieldgroup"}
      class:super-container-item={$childState == "containerItem"}
      class:accordion-item={$childState == "accordionItem"}
      class:tab-item={$childState == "activeTabsItem"}
      class:splitview-item={$childState == "splitviewItem"}
      class:super-fieldgroup-item={$childState == "fieldgroupItem"}
      class:spectrum-OpacityCheckerboard={$builderStore.inBuilder &&
        $component.empty}
      use:styleable={$component.styles}
      on:click={onClick ? onClick : () => {}}
      on:contextmenu={(e) => {
        if (onRightClick) {
          e.preventDefault();
          onRightClick();
        }
      }}
    >
      {#if !collapsed}
        {#if mode == "grid" && inBuilder}
          <div class="underlay">
            {#each coords as _, idx}
              <div class="placeholder" />
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
            {gap}
            {quietTabs}
            {tabsAlignment}
            {tabsSize}
            {tabsIconsOnly}
            {list_icon}
            {list_title}
          />
        {/if}

        {#if childMode == "tabsItem" && $parentGridStore?.theme == "list"}
          <div class="tab-title">{title}</div>
        {/if}

        {#key labelWidth}
          {#key labelPos}
            {#key disabled}
              <slot />
            {/key}
          {/key}
        {/key}

        {#if grabberPosition}
          <Grabber {grabberPosition} {resizing} {state} />
        {/if}
      {:else}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div class="collapsed-title" on:click={handleCollapse}>
          <span>{collapseTitle?.toUpperCase() || ""}</span>
        </div>
      {/if}

      <!-- svelte-ignore a11y-no-static-element-interactions -->
      {#if collapsible}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div
          class="collapser"
          class:collapsed
          class:right={collapseSide == "right"}
          class:top={collapseSide == "top"}
          on:click={handleCollapse}
        >
          {#if !collapsed}
            <span> {panelTitle?.toUpperCase() || ""}</span>
          {/if}
          <Expander {collapsed} {collapseIcon} />
        </div>
      {/if}
    </div>
  {/if}
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
    transition: background-color 0.2s ease-in-out;

    &:hover {
      &.hoverable {
        background: var(--container-hover-background) !important;
        color: var(--container-hover-color);
        border: 1px solid var(--container-hover-border);
      }

      &.clickable {
        cursor: pointer;
      }
      & > .collapsed-title {
        color: var(--spectrum-global-color-gray-700);
      }
      & > .collapser {
        color: var(--spectrum-global-color-gray-700);
      }
    }

    &.collapsible {
      border: 1px solid var(--spectrum-global-color-gray-200);
      width: 260px;
    }
    &.collapsed {
      cursor: pointer;
      background-color: var(--spectrum-global-color-gray-100);

      &:hover {
        background-color: var(--spectrum-global-color-gray-200);
        & > .collapser {
          background-color: var(--spectrum-global-color-gray-200);
          color: var(--spectrum-global-color-gray-900);
        }
      }
    }
    & > .collapser {
      position: absolute;
      top: 0px;
      left: 0px;
      right: 0px;
      height: 2.4rem;
      cursor: pointer;
      color: var(--spectrum-global-color-gray-600);
      border-bottom: 1px solid var(--spectrum-global-color-gray-200);
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 12px;
      font-weight: 600;
      padding: 0.5rem;
      z-index: 1;

      &:hover {
        color: var(--spectrum-global-color-gray-900);
      }

      &.collapsed {
        justify-content: center;
      }

      &.right {
        left: 8px;
        right: unset;
      }
      &.top {
        top: unset;
        bottom: 8px;
      }
    }

    & > .collapsed-title {
      flex: auto;
      display: flex;
      justify-content: center;
      padding: 0.75rem;
      color: var(--spectrum-global-color-gray-500);
      & > span {
        writing-mode: vertical-rl;
        text-orientation: upright;
        font-size: 16px;
        font-weight: 600;
        letter-spacing: 1.2;
      }
    }
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
    background-color: var(--spectrum-global-color-gray-75);
  }

  .placeholder {
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--spectrum-global-color-gray-500);
    border: 1px;
    border: 1px dashed var(--spectrum-global-color-gray-400);
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

  .spectrum-OpacityCheckerboard {
    block-size: unset;
    inline-size: unset;
  }

  .tab-title {
    display: flex;
    align-items: center;
    padding: 1rem 1rem;
    max-width: 100%;
    font-size: 12px;
    color: var(--spectrum-global-color-gray-800);
    border-bottom: 2px solid var(--spectrum-global-color-gray-200);
    text-transform: uppercase;
    letter-spacing: 1.2px;
    font-weight: 500;
    padding-left: 1.5rem;
  }
</style>
