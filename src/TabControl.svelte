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

  $: setTimeout(() => getIndicatorPosition($$props, tabs) , 50 ) 
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
  class:tabsVertical = { direction == "column" } 
  class:tabsHorizontal = { direction == "row" } 
  style:justify-content = { direction == "row" ? hAlign : vAlign }
  style:--tab-size={tabsSize}
  style:--tab-alignment={ tabsAlignment } 
  style:--tab-track-thickness={ "calc( 0.1 * var(--tab-size) )" }
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
                    ? "auto" : "none"
                    }
        on:click={() => state.selectTab(container.id)}
      >
        <span class="tabText">{ container.title || "Tab " + idx }</span>
      </div>
    {/each}
</div>

<style>
  .tabsHorizontal {
    position: relative;
    display: flex;
    background-color: var(--tabControlFillColor );
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
    min-width: 0;
    background-color: var(--tabControlFillColor );
  }
  .tabsVertical::before {
    position: absolute;
    top: var(--tabIndicatorTop);
    right: 0px;
    height: var(--tabIndicatorHeight);
    width: 100%;
    content: "";
    background-color: var(--tabIndicatorFillColor );
    transition: all 230ms;
    border-right: calc( 0.075 * var(--tab-size) ) solid var(--primaryColor);
    z-index: 2;
  }

  .tabsVertical::after {
    position: absolute;
    width: var(--tab-track-thickness);
    height: 100%;
    right: 0px;
    background-color: var(--tabControlFillColor );
    z-index: 1;
    content: "";
  }

  .tab {
    height: var(--tab-size);
    width: calc( 2.85 * var(--tab-size) );
    padding-left: calc( 0.25 * var(--tab-size) );
    padding-right: calc( 0.25 * var(--tab-size) );
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: var(--tab-alignment);
  }

  .tabText {
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
  }
  .tab:hover {
    cursor: pointer;
    background-color: rgba(0,0,0, 0.055);
  }

  .selectedTab {
    color: var(--spectrum-global-color-gray-900);
    font-weight: bold;
  }
</style>