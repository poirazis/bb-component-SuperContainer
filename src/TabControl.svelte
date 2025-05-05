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
  export let quietTabs;

  let tabs = [];
  let innerWidth, innerHeight;

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

  $: getIndicatorPosition(innerWidth, innerHeight);
  $: setTimeout(() => getIndicatorPosition($$props, tabs), 10);
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
{#if containers?.length}
  <div
    class="outer-tabs"
    class:quietTabs
    class:vertical={direction == "column"}
    style:justify-content={direction == "row" ? hAlign : vAlign}
  >
    <div
      class="tabs"
      bind:clientWidth={innerWidth}
      bind:clientHeight={innerHeight}
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
        : "var(--spectrum-global-color-gray-700)"}
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
              style:color={container.id == selectedTab
                ? container?.color
                : null}
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
  </div>
{/if}

<style>
  .outer-tabs {
    display: flex;
    flex-direction: row;
    width: 100%;
    overflow: hidden;
    position: relative;
    justify-content: stretch;

    &.vertical {
      flex-direction: column;
      width: var(--tab-width);
      height: 100%;
      justify-content: stretch;
      align-items: stretch;
    }

    &.quietTabs {
      justify-content: center;

      & > .tabs {
        flex: none !important;

        &.vertical {
          height: unset !important;
          justify-content: center;
        }
      }
    }
  }
  .tabs {
    flex: 1 0 auto;
    position: relative;
    display: flex;
    gap: 0.85rem;
    min-height: 2.25rem;
    margin-bottom: var(--gap);
    padding-bottom: 0.5rem;

    &.buttons {
      gap: 0;
      padding-bottom: 0rem;

      &::after {
        border-bottom: unset;
      }
    }

    &::before {
      position: absolute;
      left: var(--tabIndicatorLeft);
      height: 100%;
      bottom: 0;
      width: var(--tabIndicatorWidth);
      content: "";
      transition: all 130ms ease-out;
      border-bottom: var(--tab-track-thickness) solid var(--tab-selected-color);
    }

    &::after {
      position: absolute;
      border-bottom: var(--tab-track-thickness) solid
        var(--spectrum-global-color-gray-300);
      width: 100%;
      bottom: 0;
      content: "";
      z-index: -2;
    }

    &.vertical {
      height: 100%;
      width: var(--tab-width);
      flex-direction: column;
      gap: 0 !important;
      margin-bottom: unset;
      margin-right: var(--gap);

      &::before {
        top: var(--tabIndicatorTop);
        right: 0;
        height: var(--tabIndicatorHeight);
        transition: all 230ms;
        border: none;
        border-right: var(--tab-track-thickness) solid var(--tab-selected-color);
      }

      &::after {
        height: 100%;
        width: 100%;
        border: none;
        border-right: var(--tab-track-thickness) solid
          var(--spectrum-global-color-gray-300);
        z-index: -2;
      }

      &.buttons {
        gap: unset;
        &::after {
          border-right: unset;
        }
      }
    }
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
    height: var(--tab-height);
    max-width: var(--tab-width);

    &.button {
      padding: var(--tab-padding);
      height: var(--tab-height);
      max-width: var(--tab-width);
      align-items: center;
      justify-content: var(--tab-alignment);

      &.selected::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
        z-index: -1;
        border-radius: 5px;
        background-color: var(--spectrum-global-color-gray-300);
      }
    }

    &.vertical {
      border: none;
      height: var(--tab-height);
    }

    &:hover {
      cursor: pointer;
      color: var(--spectrum-global-color-gray-800);

      &.button:not(.selected) {
        background-color: var(--spectrum-global-color-gray-200);
        border-radius: 5px;
      }
    }

    &.selected {
      color: var(--tab-selected-color);
      font-weight: 600;

      &:hover {
        color: var(--tab-selected-color);
        cursor: default;
      }
    }
  }

  .tab-text {
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
  }
</style>
