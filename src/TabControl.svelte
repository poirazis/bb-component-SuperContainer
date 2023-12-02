<script>
  export let containers
  export let direction
  export let selectedTab
  export let hAlign
  export let vAlign
  export let state
  export let theme
  export let tabsQuiet 
  export let tabsSize
  export let tabsAlignment
  export let tabsIconsOnly
  export let tabsEmphasized
  
  let tabs = []  

  let indicatorLeft, indicatorWidth, indicatorTop, indicatorHeight

  const getIndicatorPosition = () => {
    if ( tabs[selectedTab] ) {
      indicatorLeft = tabs[selectedTab].offsetLeft
      indicatorWidth = tabs[selectedTab].clientWidth
      indicatorTop = tabs[selectedTab].offsetTop
      indicatorHeight = tabs[selectedTab].clientHeight
    }
  }

  $: setTimeout(() => getIndicatorPosition($$props, tabs) , 10 ) 
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
{#if containers?.length}
  <div
    class:tabsVertical = { direction == "column" } 
    class:tabsHorizontal = { direction == "row" } 
    style:justify-content = { direction == "row" ? hAlign : vAlign }
    style:--tab-width={ direction == "row" ? "auto" : "8rem" }
    style:--tab-height={ direction == "row" ? "3rem" : "3rem"}
    style:--tab-padding={ direction == "row" ? "0.25rem" : "0.75rem" }
    style:--tab-alignment={ tabsAlignment } 
    style:--tab-track-thickness={ "2px" }
    style:--tab-selected-color={tabsEmphasized ? "var(--primaryColor)" : "var(--spectrum-global-color-gray-900)"}
    style:--tabIndicatorLeft = { indicatorLeft } 
    style:--tabIndicatorWidth = { indicatorWidth } 
    style:--tabIndicatorTop = { indicatorTop } 
    style:--tabIndicatorHeight = { indicatorHeight } 
    style:--tabIndicatorFillColor = { theme == "budibase" ? "transparent" : "rgba(255,255,255, 0.085)"}
    style:--tabControlFillColor = { theme == "budibase" ? "transparent" : "rgba(0,0,0, 0.085)"}
    style:--tabControlTrackFillColor = { theme == "budibase" && tabsQuiet ? "transparent" : "var(--spectrum-global-color-gray-200)"}
    >
      {#each containers as container, idx }
        <div
          bind:this={tabs[container.id]} 
          class="tab"
          class:selectedTab={container.id == selectedTab}
          style:flex={ ( direction == "row" && hAlign == "stretch" )
                      || ( direction == "column" && vAlign == "stretch" ) 
                      ? "1 1 auto" : "0 0 auto"
                      }
          on:click={() => state.selectTab(container.id)}
        >
          {#if container.icon}
            <div class="tabIcon" style:color={container.id == selectedTab ? container?.color : null } style:--tab-icon-size={tabsIconsOnly ? "24px" : "20px"}>
              {@html container?.icon}
            </div>
          {/if}
          {#if !tabsIconsOnly || !container.icon}
            <span class="tabText">
              { container.title || "Tab " + idx }
            </span>
          {/if}
        </div>
      {/each}
  </div>
{/if}

<style>
  .tabsHorizontal {
    position: relative;
    display: flex;
    background-color: var(--tabControlFillColor );
    gap: 1rem;
  }
  .tabsHorizontal::before {
    position: absolute;
    left: var(--tabIndicatorLeft);
    height: 100%;
    bottom: 0;
    width: var(--tabIndicatorWidth);
    content: "";
    background-color: var(--tabIndicatorFillColor );
    color: var(--primaryColor);
    transition: all 130ms ease-out;
    border-bottom: var(--tab-track-thickness) solid var(--primaryColor);
    z-index: 2;
  }

  .tabsHorizontal::after {
    position: absolute;
    height: var(--tab-track-thickness);
    width: 100%;
    bottom: 0px;
    background-color: var(--tabControlTrackFillColor);
    z-index: 1;
    content: "";
  }

  .tabsVertical {
    display: flex;
    position: relative;
    flex-direction: column;
    align-items: stretch;
    background-color: var(--tabControlFillColor );
  }

  .tabsVertical > .selectedTab {
    margin-left: 0.25rem;
  }
  .tabsVertical::before {
    position: absolute;
    top: var(--tabIndicatorTop);
    left: 0px;
    height: var(--tabIndicatorHeight);
    content: "";
    background-color: var(--tabIndicatorFillColor );
    transition: all 230ms;
    border-right: var(--tab-track-thickness) solid var(--tab-selected-color);
    z-index: 2;
  }

  .tabsVertical::after {
    position: absolute;
    width: var(--tab-track-thickness);
    height: 100%;
    left: 0px;
    background-color: var(--tabControlTrackFillColor);
    z-index: 1;
    content: "";
  }

  .tab {
    height: var(--tab-height);
    width: var(--tab-width);
    padding-left: var(--tab-padding);
    padding-right: var(--tab-padding);
    box-sizing: border-box;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    justify-content: var(--tab-alignment);
    color: var(--spectrum-global-color-gray-600);
  }

  .tabText {
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
  }

  .tabIcon {
    display: grid;
    align-items: center;
  }

  :global( .tabIcon > svg ) {
    height: var(--tab-icon-size);
  }

  .tab:hover {
    cursor: pointer;
    color: var(--spectrum-global-color-gray-800);
  }

  .selectedTab {
    color: var(--spectrum-global-color-gray-900);
  }
</style>