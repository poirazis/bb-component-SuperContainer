<script>
  import { getContext, onDestroy, onMount, setContext, untrack } from "svelte";
  import { SuperTabs } from "@poirazis/superlib";
  import FieldGroupProvider from "../lib/FieldGroupProvider.svelte";

  const { styleable, builderStore } = getContext("sdk");
  const component = getContext("component");

  const parentState = getContext("superContainer");
  const getParentParams = getContext("superContainerParams");
  let parentParams = $derived(getParentParams?.() ?? null);

  let {
    children,
    flex,
    flexFactor = 1,
    mode = "container",
    childMode = "containerItem",
    direction,
    hAlign = "stretch",
    vAlign,
    gap,
    wrap,
    tabsPosition = "top",
    tabsWidth = "200px",
    tabsAlignment,
    buttonsAlignment = "flex-start",
    activeTab = 0,
    tabsIconsOnly = false,
    theme = "budibase",
    list_title = "Settings",
    list_icon = "gear",
    tabDisabled,
    isTabSection = false,
    listBackground,
    hiddenTabs = "hidden",
    gridColumns = 3,
    gridRows = 3,
    gridAutoRows = "auto",
    gridAutoColumns = "auto",
    gridJustifyItems = "stretch",
    gridAlignItems = "stretch",
    gridJustifyContent = "start",
    gridAlignContent = "start",
    title,
    icon,
    color,
    labelPos,
    labelWidth = "6rem",
    fgcolumnGap,
    fgrowGap,
    disabled = false,
    readonly = false,
    colSpan = 1,
    rowSpan = 1,
    gridColumnStart,
    gridColumnEnd,
    gridRowStart,
    gridRowEnd,
    alignSelf = "auto",
    justifySelf = "auto",
    gridZIndex,
    onTabChange,
    onClick,
    onRightClick,
    hoverBackground,
    hoverBorder,
    hoverText,
    onShow,
  } = $props();

  let containers = $state([]);
  let selectedTab = $state(undefined);
  let tabChangeInitialized = $state(false);
  let forceHidden = $state(false);

  let componentID = $derived($component.id);
  let effectiveIcon = $derived(icon ? "ph ph-" + icon : null);
  let inBuilder = $derived($builderStore.inBuilder);
  let selected = $derived($component.selected || $component.inSelectedPath);
  let refreshKey = $derived(mode + labelPos + labelWidth + hiddenTabs);

  function resolveChildMode(parentLayout) {
    if (!parentLayout || parentLayout == "container") {
      return "containerItem";
    }
    return parentLayout + "Item";
  }

  let effectiveChildMode = $derived(
    parentParams ? resolveChildMode(parentParams.layout) : childMode,
  );

  let isActiveTab = $derived(
    effectiveChildMode == "tabsItem" &&
      parentParams?.selectedTab == componentID,
  );

  let shouldRender = $derived.by(() => {
    if (forceHidden) return false;
    if (effectiveChildMode != "tabsItem") return true;
    if (parentParams?.hiddenTabs == false) return isActiveTab;
    return true;
  });

  let childCssVariables = $derived.by(() => {
    const gridColumn = gridColumnStart
      ? gridColumnStart + " / " + (gridColumnEnd || "span 1")
      : null;
    const gridRow = gridRowStart
      ? gridRowStart + " / " + (gridRowEnd || "span 1")
      : null;

    switch (effectiveChildMode) {
      case "containerItem":
        return {
          flex: flex == "grow" ? flexFactor : "none",
        };
      case "panelItem":
        return {
          flex: flex == "grow" ? flexFactor + " 1 auto" : "0 0 auto",
        };
      case "gridItem":
        if (parentParams) {
          return {
            "grid-column": gridColumn
              ? gridColumn
              : "span " + Math.min(colSpan, parentParams.gridColumns),
            "grid-row": gridRow
              ? gridRow
              : "span " + Math.min(rowSpan, parentParams.gridRows),
            "align-self": alignSelf,
            "justify-self": justifySelf,
            ...(gridZIndex && { "z-index": gridZIndex }),
            override: "hidden",
          };
        }
        return {
          "grid-column": gridColumn ? gridColumn : "span " + colSpan * 6,
          "grid-row": gridRow ? gridRow : "span " + rowSpan,
          "align-self": alignSelf,
          "justify-self": justifySelf,
          ...(gridZIndex && { "z-index": gridZIndex }),
        };
      case "tabsItem":
        return {
          flex: "auto",
        };
      case "fieldgroupItem":
        return {
          fieldgroupitem: "yes",
          "grid-column": gridColumn ? gridColumn : "span " + colSpan * 6,
          "grid-row": gridRow ? gridRow : "span " + rowSpan,
          "align-self": alignSelf,
          "justify-self": justifySelf,
          ...(gridZIndex && { "z-index": gridZIndex }),
        };
      default:
        return {};
    }
  });

  let containerParams = $derived.by(() => ({
    layout: mode,
    gridColumns,
    gridRows,
    selectedTab: mode === "tabs" ? selectedTab : undefined,
    hiddenTabs,
    theme,
  }));

  const superContainer = {
    registerContainer(componentId, title, icon, color, disabled, isTabSection) {
      if (containers.some((entry) => entry.id == componentId)) return;

      containers = [
        ...containers,
        {
          id: componentId,
          title,
          icon,
          color,
          disabled,
          isTabSection,
        },
      ];
    },
    updateContainer(
      componentId,
      title,
      icon,
      color,
      tabDisabled,
      isTabSection,
    ) {
      const index = containers.findIndex((e) => e.id == componentId);
      if (index === -1) return;

      const entry = containers[index];
      if (
        entry.title === title &&
        entry.icon === icon &&
        entry.color === color &&
        entry.disabled === tabDisabled &&
        entry.isTabSection === isTabSection
      ) {
        return;
      }

      containers = containers.map((item, i) =>
        i === index
          ? {
              ...item,
              title,
              icon,
              color,
              disabled: tabDisabled,
              isTabSection,
            }
          : item,
      );
    },
    unregisterContainer(componentId) {
      const index = containers.findIndex((e) => e.id == componentId);
      if (index === -1) return;

      containers = containers.filter((e) => e.id != componentId);

      if (mode == "tabs" && containers.length > 0) {
        this.selectTab(containers[0].id);
      }
    },
    selectTab(tabId) {
      if (mode !== "tabs" || tabId == selectedTab) return;

      selectedTab = tabId;
      if (tabChangeInitialized) {
        onTabChange?.({ tabTitle: title });
      }
      tabChangeInitialized = true;
    },
    selectChild(componentId) {
      if (mode !== "tabs") return;
      if (containers.some((entry) => entry.id == componentId)) {
        this.selectTab(componentId);
      }
    },
    hide() {
      forceHidden = true;
    },
    show() {
      forceHidden = false;
      onShow?.();
    },
  };

  let cssVariables = $derived.by(() => {
    switch (mode) {
      case "container":
        return {
          "flex-direction": direction,
          "flex-wrap": wrap ? "wrap" : "nowrap",
          "justify-content": direction == "row" ? hAlign : vAlign,
          "align-items": direction == "row" ? vAlign : hAlign,
          "align-content": wrap ? (direction == "row" ? vAlign : hAlign) : null,
          gap: gap + "rem",
        };
      case "grid":
        return {
          display: "grid",
          "grid-template-columns":
            gridColumns > 0 ? `repeat(${gridColumns}, 1fr)` : "auto",
          "grid-template-rows":
            gridRows > 0 ? `repeat(${gridRows}, 1fr)` : "auto",
          "grid-auto-rows": gridAutoRows,
          "grid-auto-columns": gridAutoColumns,
          "justify-items": gridJustifyItems,
          "align-items": gridAlignItems,
          "justify-content": gridJustifyContent,
          "align-content": gridAlignContent,
          "column-gap": gap ? gap + "rem" : null,
          "row-gap": gap ? gap + "rem" : null,
          "--grid-columns": gridColumns,
          "--grid-column-gap": gap + "rem",
          "--grid-row-gap": gap + "rem",
          "--grid-content-align": gridAlignContent,
        };
      case "tabs":
        return {
          gap: theme == "list" ? "0rem" : gap,
          "flex-direction":
            tabsPosition == "left" || theme == "list" ? "row" : "column",
        };
      case "fieldgroup":
        return {
          display: "grid",
          "justify-items": "stretch",
          "align-items": "stretch",
          "align-content": "start",
          "--grid-columns": gridColumns * 6,
          "--field-group-column-gap": fgcolumnGap || "0.5rem",
          "--field-group-row-gap": fgrowGap || "0.25rem",
        };
      default:
        return {};
    }
  });

  let enrichedStyles = $derived({
    ...$component.styles,
    normal: {
      ...$component.styles?.normal,
      ...cssVariables,
      ...childCssVariables,
      "--container-hover-background": hoverBackground,
      "--container-hover-border": hoverBorder,
      "--container-hover-color": hoverText,
    },
  });

  $effect(() => {
    if (mode !== "tabs") {
      selectedTab = undefined;
    }
  });

  $effect(() => {
    if (mode !== "tabs" || containers.length === 0) return;

    const index = Number(activeTab);
    if (!Number.isFinite(index) || index < 0 || index >= containers.length) {
      return;
    }

    const tabId = containers[index]?.id;
    if (tabId == null) return;

    untrack(() => {
      if (selectedTab !== tabId) {
        superContainer.selectTab(tabId);
      }
    });
  });

  let wasActiveTab = false;

  $effect(() => {
    if (effectiveChildMode != "tabsItem") {
      wasActiveTab = false;
      return;
    }

    if (isActiveTab && !wasActiveTab) {
      onShow?.();
    }

    wasActiveTab = isActiveTab;
  });

  $effect(() => {
    parentState?.updateContainer(
      componentID,
      title,
      effectiveIcon,
      color,
      tabDisabled,
      isTabSection,
    );
  });

  $effect(() => {
    if (inBuilder && selected && parentState) {
      parentState.selectChild($component.id);
      if (childMode != parentParams.layout + "Item") {
        builderStore.actions.updateProp(
          "childMode",
          parentParams.layout + "Item",
        );
      }
    } else if (
      inBuilder &&
      selected &&
      !parentState &&
      childMode != "containerItem"
    ) {
      builderStore.actions.updateProp("childMode", "containerItem");
    }
  });

  onMount(() => {
    parentState?.registerContainer(
      componentID,
      title,
      effectiveIcon,
      color,
      tabDisabled,
      isTabSection,
    );
  });

  setContext("superContainer", superContainer);
  setContext("superContainerParams", () => containerParams);

  onDestroy(() => {
    parentState?.unregisterContainer(componentID);
  });
</script>

{#key refreshKey}
  <!-- svelte-ignore event_directive_deprecated -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  {#if shouldRender}
    <div
      class:hoverable={hoverBackground || hoverBorder || hoverText}
      class:clickable={onClick}
      class:super-container={mode == "container"}
      class:super-grid={mode == "grid"}
      class:tabs={mode == "tabs"}
      class:super-fieldgroup={mode == "fieldgroup"}
      class:super-container-item={effectiveChildMode == "containerItem"}
      class:tab-item={effectiveChildMode == "tabsItem" && isActiveTab}
      class:tab-item-hidden={effectiveChildMode == "tabsItem" && !isActiveTab}
      class:super-fieldgroup-item={effectiveChildMode == "fieldgroupItem"}
      class:in-builder={inBuilder}
      use:styleable={enrichedStyles}
      on:click={onClick ? onClick : () => {}}
      on:contextmenu={(e) => {
        if (onRightClick) {
          e.preventDefault();
          onRightClick();
        }
      }}
    >
      {#if mode == "tabs" && containers?.length > 0}
        <SuperTabs
          {containers}
          {selectedTab}
          {direction}
          {theme}
          {tabsPosition}
          {tabsWidth}
          {tabsAlignment}
          {buttonsAlignment}
          {tabsIconsOnly}
          {list_icon}
          {list_title}
          {listBackground}
          on:change={(e) => {
            superContainer.selectTab(e.detail.id);
          }}
        />
      {/if}

      {#if mode == "fieldgroup"}
        {#key `${labelPos}-${gridColumns}-${labelWidth}-${disabled}-${readonly}`}
          <FieldGroupProvider
            {labelPos}
            {gridColumns}
            {labelWidth}
            {disabled}
            {readonly}
          >
            {@render children?.()}
          </FieldGroupProvider>
        {/key}
      {:else}
        {@render children?.()}
      {/if}
    </div>
  {/if}
{/key}

<style>
  :global(.super-grid > .component > *) {
    overflow: hidden;
  }

  .super-container {
    flex-shrink: 1;
    display: flex;
    overflow: hidden;
    transition: background-color 0.2s ease-in-out;
    min-width: 0;

    &:hover {
      &.hoverable {
        background: var(--container-hover-background) !important;
        color: var(--container-hover-color);
        border: 1px solid var(--container-hover-border);
      }

      &.clickable {
        cursor: pointer;
      }
    }
  }
  .super-container-item {
    flex: 1;
    min-width: 0;
  }
  .super-grid {
    display: grid;
    position: relative;
    overflow: hidden;
    grid-template-columns: repeat(var(--grid-columns), 1fr);
    grid-template-rows: var(--grid-rows);
    column-gap: var(--grid-column-gap);
    row-gap: var(--grid-row-gap);
    align-content: var(--grid-content-align);
  }

  .super-fieldgroup {
    display: grid;
    position: relative;
    grid-template-columns: repeat(var(--grid-columns), 1fr);
    column-gap: var(--field-group-column-gap, 0.5rem);
    row-gap: var(--field-group-row-gap, 0.5rem);
  }

  :global(.super-fieldgroup > .component > *) {
    grid-column: span 6;
  }

  .tabs {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    justify-content: stretch;
    overflow: hidden;
  }

  .tab-item {
    display: flex;
    overflow: hidden;
  }

  .tab-item-hidden {
    display: none !important;
  }
</style>
