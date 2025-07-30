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
  export let list_icon;
  export let list_title;

  export let quietTabs;

  let tabs = [];
  let innerWidth, innerHeight;

  let indicatorLeft, indicatorWidth, indicatorTop, indicatorHeight;

  const tabWidths = {
    S: "7.5rem",
    M: "8rem",
    L: "10rem",
  };

  const tabPaddings = {
    S: "0.25rem 0.5rem",
    M: "0.35rem 0.75rem",
    L: "0.75rem 1rem",
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
  $: setTimeout(() => getIndicatorPosition($$props, tabs), 50);
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
{#if containers?.length}
  <div
    class="outer-tabs"
    class:quietTabs
    class:list={theme == "list"}
    class:vertical={direction == "column"}
    style:justify-content={direction == "row" ? hAlign : vAlign}
  >
    <div
      class="tabs"
      bind:clientWidth={innerWidth}
      bind:clientHeight={innerHeight}
      class:vertical={direction == "column" || theme == "list"}
      class:buttons={theme == "buttons"}
      class:list={theme == "list"}
      style:justify-content={direction == "row" ? hAlign : vAlign}
      style:--tab-width={tabWidths[tabsSize]}
      style:--tab-padding={tabPaddings[tabsSize]}
      style:--tab-alignment={tabsAlignment}
      style:--tab-track-thickness={"2px"}
      style:--tabIndicatorLeft={indicatorLeft}
      style:--tabIndicatorWidth={indicatorWidth}
      style:--tabIndicatorTop={indicatorTop}
      style:--tabIndicatorHeight={indicatorHeight}
      style:--tab-selected-color="var(--spectrum-global-color-gray-800)"
      style:--gap={gap + "rem"}
    >
      {#if theme == "list" && list_title}
        <div class="tab list-title">
          {#if list_icon}
            <i class={list_icon} />
          {/if}
          {list_title}
        </div>
      {/if}
      {#each containers as container, idx (idx)}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div
          bind:this={tabs[container.id]}
          class="tab"
          class:vertical={direction == "column"}
          class:button={theme == "buttons"}
          class:list={theme == "list"}
          class:selected={container.id == selectedTab}
          class:disabled={container.disabled}
          on:click={() => {
            if (!container.disabled) state.selectTab(container.id);
          }}
        >
          {#if container.icon}
            <i
              class={container.icon}
              style:font-size={tabsIconsOnly ? "20px" : null}
              style:color={container.color}
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
      align-items: stretch;
    }

    &.list {
      width: 15rem;
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
    gap: 0.5rem;
    min-height: 2.25rem;
    margin-bottom: var(--gap);
    padding-bottom: 0.5rem;

    &.buttons {
      gap: 0.25rem;
      padding-bottom: 0.75rem;
    }

    &.list {
      min-width: 15rem !important;
      gap: 0rem;
      background-color: var(--spectrum-global-color-gray-50);
      border-right: unset;
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
        var(--spectrum-global-color-gray-200);
      width: 100%;
      bottom: 0;
      content: "";
      z-index: -2;
    }

    &.vertical {
      height: 100%;
      width: var(--tab-width);
      flex-direction: column;
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
        gap: 0.25rem;
        padding-right: 0.5rem;
        &::after {
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
      max-width: var(--tab-width);
      align-items: center;
      justify-content: var(--tab-alignment);
      min-height: unset;
      border-radius: 4px;

      &.selected {
        background-color: var(--spectrum-global-color-gray-200);
      }
    }

    &.list {
      display: flex;
      align-items: center;
      padding: 0.5rem 1rem;
      max-width: 100%;
      font-size: 13px;
      color: var(--spectrum-global-color-gray-700);
      font-weight: 400;

      &.selected {
        color: var(--tab-selected-color);
        background-color: var(--spectrum-global-color-gray-200) !important;
        font-weight: 500;
      }

      &:hover:not(.disabled) {
        background-color: var(--spectrum-global-color-gray-75);
      }

      &.disabled {
        color: var(--spectrum-global-color-gray-500);
      }
    }

    &.list-title {
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
    }

    &.vertical {
      border: none;
    }

    &:hover:not(.disabled):not(.list-title) {
      cursor: pointer;
      color: var(--spectrum-global-color-gray-800);

      &.button:not(.selected) {
        background-color: var(--spectrum-global-color-gray-100);
      }
    }

    &.selected {
      color: var(--tab-selected-color);

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
