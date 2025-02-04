<script>
  export let containers;
  export let direction;
  export let selectedTab;
  export let hAlign;
  export let vAlign;
  export let state;
  export let theme;
  export let gap;
  export let tabsSize;
  export let tabsAlignment;
  export let tabsIconsOnly;
  export let tabsEmphasized;

  let tabs = [];
  let innerWidth;

  let indicatorLeft, indicatorWidth, indicatorTop, indicatorHeight;

  const tabHeights = {
    S: "2rem",
    M: "2.25rem",
    L: "2.5rem",
  };
  const tabWidths = {
    S: "7.5rem",
    M: "8rem",
    L: "10rem",
  };
  const tabPaddings = {
    S: "0.85rem",
    M: "1rem",
    L: "1.25rem",
  };

  const getIndicatorPosition = () => {
    if (tabs[selectedTab] && theme == "budibase") {
      indicatorLeft = tabs[selectedTab].offsetLeft;
      indicatorWidth = tabs[selectedTab].clientWidth;
      indicatorTop = tabs[selectedTab].offsetTop;
      indicatorHeight = tabs[selectedTab].clientHeight;
    } else {
      indicatorLeft = null;
      indicatorWidth = null;
      indicatorTop = null;
      indicatorHeight = null;
    }
  };

  $: getIndicatorPosition(innerWidth);
  $: setTimeout(() => getIndicatorPosition($$props, tabs), 10);
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
{#if containers?.length}
  <div
    class="tabs"
    bind:clientWidth={innerWidth}
    class:vertical={direction == "column"}
    class:buttons={theme == "buttons"}
    style:justify-content={direction == "row" ? hAlign : vAlign}
    style:--tab-width={tabWidths[tabsSize]}
    style:--tab-height={tabHeights[tabsSize]}
    style:--tab-padding={tabPaddings[tabsSize]}
    style:--tab-alignment={tabsAlignment}
    style:--tab-track-thickness={"2px"}
    style:--tab-selected-color={tabsEmphasized
      ? "var(--primaryColor)"
      : "var(--spectrum-global-color-gray-800)"}
    style:--tabIndicatorLeft={indicatorLeft}
    style:--tabIndicatorWidth={indicatorWidth}
    style:--tabIndicatorTop={indicatorTop}
    style:--tabIndicatorHeight={indicatorHeight}
    style:--gap={gap + "rem"}
  >
    {#each containers as container, idx (idx)}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <div
        bind:this={tabs[container.id]}
        class="tab"
        class:vertical={direction == "column"}
        class:button={theme == "buttons"}
        class:selected={container.id == selectedTab}
        on:click={() => state.selectTab(container.id)}
      >
        {#if container.icon}
          <i
            class={container.icon}
            style:font-size={tabsIconsOnly ? "20px" : null}
            style:color={container.id == selectedTab ? container?.color : null}
          />
        {/if}

        {#if !tabsIconsOnly || !container.icon}
          <span class="tab-text">
            {container.title || "Tab " + idx}
          </span>
        {/if}
      </div>
    {/each}
  </div>
{/if}

<style>
  .tabs {
    position: relative;
    display: flex;
    gap: 0.85rem;
    min-height: 2.4rem;
    margin-bottom: var(--gap);
  }
  .tabs.buttons {
    position: relative;
    display: flex;
    gap: 0rem;
  }

  .tabs::before {
    position: absolute;
    left: var(--tabIndicatorLeft);
    height: 100%;
    bottom: 0;
    width: var(--tabIndicatorWidth);
    content: "";
    transition: all 130ms ease-out;
    border-bottom: var(--tab-track-thickness) solid var(--tab-selected-color);
  }

  .tabs::after {
    position: absolute;
    border-bottom: var(--tab-track-thickness) solid
      var(--spectrum-global-color-gray-300);
    width: 100%;
    bottom: 0px;
    content: "";
    z-index: -2;
  }
  .tabs.vertical {
    position: relative;
    height: 100%;
    width: var(--tab-width);
    flex-direction: column;
    gap: 0px !important;
    margin-bottom: unset;
    margin-right: var(--gap);
  }
  .tabs.vertical::before {
    position: absolute;
    top: var(--tabIndicatorTop);
    right: 0px;
    height: var(--tabIndicatorHeight);
    content: "";
    transition: all 230ms;
    border: none;
    border-right: var(--tab-track-thickness) solid var(--tab-selected-color);
  }
  .tabs.vertical::after {
    position: absolute;
    height: 100%;
    width: 100%;
    border: none;
    border-right: var(--tab-track-thickness) solid
      var(--spectrum-global-color-gray-300);
    content: "";
    z-index: -2;
  }

  .tabs.vertical.buttons {
    gap: unset;
  }

  .tab {
    position: relative;
    box-sizing: border-box;
    padding: var(--tab-padding);
    display: flex;
    align-items: center;
    gap: 0.5rem;
    justify-content: var(--tab-alignment);
    color: var(--spectrum-global-color-gray-600);
    font-weight: 600;
    height: var(--tab-height);
    min-width: var(--tab-width);
  }

  .tab.button {
    padding: var(--tab-padding);
    height: var(--tab-height);
    width: var(--tab-width);
    align-items: center;
    justify-content: var(--tab-alignment);
  }

  .tab.selected.button::before {
    content: "";
    position: absolute;
    top: 0px;
    left: 0px;
    bottom: 0px;
    right: 0px;
    z-index: -1;
    border-radius: 5px;
    background-color: var(--spectrum-global-color-gray-200);
  }
  .tab.vertical.selected.button::before {
    content: "";
    position: absolute;
    top: 0px;
    left: 4px;
    bottom: 0px;
    right: 8px;
    z-index: -1;
    border-radius: 5px;
    background-color: var(--spectrum-global-color-gray-200);
  }

  .tab:hover {
    cursor: pointer;
    color: var(--spectrum-global-color-gray-800);
  }
  .tab.vertical {
    border: none;
    padding-left: 0.85rem;
    height: var(--tab-height);
  }

  .tab-text {
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
  }
  .selected {
    color: var(--tab-selected-color);
  }
  .selected:hover {
    color: var(--tab-selected-color);
  }
</style>
