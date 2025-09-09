<script>
  export let containers;
  export let direction;
  export let selectedTab;
  export let hAlign;
  export let vAlign;
  export let state;
  export let theme;
  export let gap;
  export let tabsAlignment;
  export let tabsIconsOnly;
  export let list_icon;
  export let list_title;

  export let quietTabs;

  let tabs = [];
  let innerWidth, innerHeight;
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
      class:negButtons={theme == "negButtons"}
      class:list={theme == "list"}
      style:justify-content={direction == "row" ? hAlign : vAlign}
      style:--tab-alignment={tabsAlignment}
      style:--tab-track-thickness="1px"
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
          class:negButtons={theme == "negButtons"}
          class:list={theme == "list"}
          class:selected={container.id == selectedTab}
          class:disabled={container.disabled}
          class:list-section={container.isTabSection}
          on:click={() => {
            if (!container.disabled && !container.isTabSection)
              state.selectTab(container.id);
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
    flex: none;
    display: flex;
    flex-direction: row;
    width: 100%;
    overflow: hidden;
    position: relative;
    justify-content: stretch;
    --selected-tab: var(--spectrum-global-color-gray-200);
    margin-bottom: 0.5rem;

    &.vertical {
      flex-direction: column;
      width: 10rem;
      align-items: stretch;
      margin-right: 0.5rem;
    }

    &.list {
      width: 15rem;
    }
  }
  .tabs {
    flex: 1 0 auto;
    position: relative;
    display: flex;
    gap: 1rem;
    height: 2.4rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid var(--spectrum-global-color-gray-200);

    &.buttons {
      gap: 0.25rem;
      padding: 0.35rem 0.25rem;
      font-size: 12px;
    }
    &.negButtons {
      gap: 0.25rem;
      padding: 0.35rem 0.25rem;
      font-size: 12px;
      background-color: var(--spectrum-global-color-gray-100);
      border-bottom: unset;
    }

    &.list {
      gap: 0rem;
      background-color: var(--spectrum-global-color-gray-50);
      border: unset;
    }

    &.vertical {
      height: 100%;
      flex-direction: column;

      border-bottom: unset;

      &.list {
        border-right: unset;
      }

      &.buttons {
        gap: 0.25rem;
        padding-right: 0.5rem;
        border-right: 1px solid var(--spectrum-global-color-gray-300);
      }
      &.negButtons {
        gap: 0.25rem;
        padding-right: 0.5rem;
      }
    }
  }

  .tab {
    position: relative;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    justify-content: var(--tab-alignment);
    color: var(--spectrum-global-color-gray-600);
    min-width: 6rem;
    font-weight: 500;

    &.disabled {
      color: var(--spectrum-global-color-gray-400) !important;
      &:hover {
        cursor: not-allowed;
      }
    }

    &.button {
      align-items: center;
      justify-content: var(--tab-alignment);
      border-radius: 4px;
      padding: 0.25rem 0.75rem;
      border: 1px solid transparent;

      &:active:not(.list-section):not(.disabled) {
        border: 1px solid var(--spectrum-global-color-gray-300);
      }
      &.selected {
        background-color: var(--selected-tab);
        border: 1px solid var(--spectrum-global-color-gray-300);
      }
    }
    &.negButtons {
      align-items: center;
      justify-content: var(--tab-alignment);
      border-radius: 4px;
      padding: 0.25rem 0.75rem;

      &:hover {
        background-color: var(--spectrum-global-color-gray-75);
      }
      &.selected {
        background-color: var(--spectrum-global-color-gray-50);
        border: 1px solid var(--spectrum-global-color-gray-300);
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
        background-color: var(--selected-tab) !important;
        font-weight: 500;
      }

      &:hover:not(.disabled):not(.list-section) {
        background-color: var(--spectrum-global-color-gray-75);
      }
    }

    &.list-title {
      display: flex;
      align-items: center;
      padding: 0.75rem 1rem;
      max-width: 100%;
      font-size: 12px;
      color: var(--spectrum-global-color-gray-800);
      text-transform: uppercase;
      letter-spacing: 1.2px;
      font-weight: 500;
      border-bottom: 1px solid var(--spectrum-global-color-gray-300);
      height: 3rem;
    }

    &.list-section {
      text-transform: uppercase;
      font-size: 11px;
      font-weight: 400;
      letter-spacing: 1.2px;
      background-color: transparent;

      &.vertical {
        margin-top: 12px;
      }

      &:hover {
        cursor: default;
      }
    }

    &.vertical {
      border: none;
      min-height: 2rem;
    }

    &:hover:not(.disabled):not(.list-title):not(.list-section) {
      cursor: pointer;
      color: var(--spectrum-global-color-gray-800);

      &.button:not(.selected) {
        background-color: var(--spectrum-global-color-gray-100);
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
