<script>
  import { getContext, onDestroy, onMount, setContext } from "svelte"
  import "@spectrum-css/opacitycheckerboard/dist/index-vars.css"
  import RepeaterPreview from "./RepeaterPreview.svelte";
  import TabControl from "./TabControl.svelte";
  import Grabber from "./Grabber.svelte";

  import fsm from "svelte-fsm"
  import { v4 as uuidv4 } from "uuid";

  const { styleable, builderStore, Provider, componentStore } = getContext("sdk")
  const component = getContext("component")
  const parentState = getContext("superLayoutManager")
   
  export let dataprovider
  export let bound
  export let flex
  export let flexFactor = 1
  export let mode = "container"
  export let direction
  export let hAlign
  export let vAlign
  export let gap
  export let wrap
  
  export let tabsSize
  export let tabsAlignment
  export let tabsQuiet

  export let title, icon, color 

  export let onClick

  let containers = []

  let container
  let managedFlexGrow
  let managedFlexShrink
  let resizing
  let startPointX
  let startPointY
  let width, height
  let initialWidth
  let initialHeight
  let grabberPosition
  let selectedTab = undefined
  let repeatableItem

  let componentID = $component.id
  let id = uuidv4()

  const state = fsm ( mode , {
    "*" : {
      registerContainer(componentID, id, state, title, icon, color ) {         
        containers = [ ...containers, {componentID: componentID, id : id, state: state, title: title, icon: icon, color: color  } ] 
        if ( mode == "tabs" && selectedTab ) state.hide(); 
        if ( mode == "tabs" ) state.setGrow(true);
        if ( mode == "tabs" && selectedTab == undefined ) selectedTab = id;
      },
      updateContainer ( id , state, title, icon , color ) {
        let index = containers.findIndex( (e) => e.id == id )
        if ( index > -1 ) {
          containers[index].state = state
          containers[index].title = title
          containers[index].icon = icon
          containers[index].color = color
        }
        containers = containers
      },
      unregisterContainer( id ) { 
        let index = containers.findIndex( (e) => e.id == id )
        if ( index > -1 ) {
          if ( selectedTab == id ) {
            selectedTab = undefined
          }
          containers.splice(index,1)
          containers = containers
        }
      },
      setMode ( mode ) {
        this.setResizingGrabbers();
        return mode 
      },
      refresh() { 
        this.refreshTabs( selectedTab );
      },
      selectChild( compID ) { 
        if ( mode == "tabs" ) {
          containers.forEach( ( { componentID, id, state } ) => { 
            if ( componentID == compID ) {
              this.selectTab( id )
            }
          } )
        }
       },
      setGrow ( bool ) { bool ? managedFlexGrow = "1" : managedFlexGrow = undefined },
      setShrink () { managedFlexShrink = "0"},
      setFixedWidth () {},
      setFixedHeight () {},
      setResizing ( pos ) { grabberPosition = pos; },
      startResizing( e ) { 
        resizing = true 
        startPointX = e.clientX
        startPointY = e.clientY
        initialWidth = container.clientWidth
        initialHeight = container.clientHeight
      },
      resize( e ) {  
        if ( grabberPosition == "right" ) {
          width = initialWidth + ( e.clientX - startPointX )
        } else {
          height = initialHeight + ( e.clientY - startPointY )
        }
      },
      stopResizing( e ) { resizing = false },
      hide() { return "hidden" },
      show() { return mode }
    },
    "hidden" : { },
    "disabled" : {},
    "container" : { 
    },
    "grid" : { },
    "splitview" : {
      _enter() {
        this.setResizingGrabbers.debounce( 100 )
      },
      setResizingGrabbers() {         
        containers.forEach( ( { state } , idx ) => { 
          state.show() 
          state.setGrow( true )
          if ( direction == "row" && idx < containers.length - 1) 
            state.setResizing ( "right" )
          else if ( direction == "column" && idx < containers.length - 1) 
            state.setResizing ( "bottom" )
        }) },
      _exit() {
        containers.forEach( ( { state } ) => { 
          state.setGrow ( false )
          state.setResizing( null ) }) 
      }
    },
    "accordion" : {

    },
    "tabs" : {
      _enter() { 
        containers.forEach( ( { id, state }, idx  ) => {
          if ( idx == 0 ) this.selectTab(id);
          state.hide(); 
          state.setGrow(true) 
        } )
      },
      _exit() { 
        selectedTab == undefined; 
        containers.forEach( ( { state } ) => { state.show(); state.setGrow(false) } )},
      selectTab ( tabId ) { 
        containers.forEach( ( {id, state } ) => { 
          if ( tabId == id ) {
            state.show();
            selectedTab = id;
          } else {
            state.hide();
          }
        } )
      },
    }
  })

  $: randomColor = bound ? "32CD3230" : Math.floor(Math.random()*16777215).toString(16) + "50";
  $: nested = component ? $component.ancestors[$component.ancestors.length - 2] == "plugin/bb-component-SuperContainer" : false
  $: state.setMode ( mode )
  $: if ( parentState ) parentState.updateContainer( id, state, title, icon, color )
//  $: if ( containers.length > 0 && mode == "tabs" && selectedTab == undefined ) state.selectTab(containers[0].id)
  $: {
    if (
      $builderStore.inBuilder && parentState
      && $componentStore.selectedComponentPath?.includes($component.id)
    ) {
      parentState.selectChild($component.id)
    }
  }


  $: $component.styles = { 
    ...$component.styles,
    normal: { 
      ...$component.styles.normal,
      "flex-grow": managedFlexGrow ? managedFlexGrow : flex == "grow" ? flexFactor : "0",
      "justify-content": mode == "container" ? direction == "row" ? hAlign : vAlign : "stretch",
      "align-items": mode == "container" ? direction == "row" ? vAlign : hAlign : "stretch",
      "display": "flex",
      "flex-direction": mode == "tabs" ? direction == "column" ? "row" : "column" : direction,
      "gap": mode != "splitview" ? gap : "0",
      "flex-wrap" : wrap ? "wrap" : "nowrap",
      "align-content": wrap ? direction == "row" ? vAlign : hAlign : null,
      "min-width" : width ?? $component.styles.normal.width ?? null,
      "max-width" : width ?? $component.styles.normal.width ?? null,
      "min-height": height ?? $component.styles.normal.height ?? null,
      "max-height": height ?? $component.styles.normal.height ?? null,
      "overflow" : "auto",
      "--random-color" : "#" + randomColor,
      "--tab-size": mode == "tabs" ? tabsSize : null,
      "--tab-alignment": mode == "tabs" ? tabsAlignment : null,
      "--tab-track-thickness": mode == "tabs" ? tabsQuiet ? "0px" : "calc( 0.05 * var(--tab-size) )" : null,
      "--spectrum-opacity-checkerboard-square-dark": "var(--spectrum-global-color-gray-50)",
      "--spectrum-opacity-checkerboard-square-light": bound ? "var(--random-color)" : "var(--spectrum-global-color-gray-75)",
      "--spectrum-opacity-checkerboard-square-size": "8px",
      "--spectrum-opacity-checkerboard-position": "left top"
    }
   }

  onMount(() => {
    if ( parentState && nested ) {
      parentState.registerContainer(componentID, id, state, title, icon, color )
    }
  })

  onDestroy(() => { 
    if ( parentState && nested ) {
      parentState.unregisterContainer( id )
    }
  })

  setContext( "superLayoutManager", state )
</script>

<svelte:window 
  on:mouseup={state.stopResizing} 
  on:mousemove={ (e) => resizing ? state.resize ( e ) : null }
  />

  {#if $state != "hidden"}
    <div 
      bind:this={container}
      class="superContainer"
      class:nested={$builderStore.inBuilder && nested}
      class:spectrum-OpacityCheckerboard={$builderStore.inBuilder}
      class:grid={mode == "grid" }
      use:styleable={$component.styles}
      >

      {#if mode == "tabs"}
        <TabControl 
          {containers}
          {hAlign}
          {vAlign}
          {direction}
          {selectedTab}
          {state}
        />
      {/if}

      {#if bound && dataprovider }
        <RepeaterPreview 
          inBuilder={$builderStore.inBuilder} 
          {mode}
          />
        {#each dataprovider.rows as row}
          <Provider data={row}>
            <slot />
          </Provider>
        {/each}
      {:else}
        <slot />
      {/if}

      {#if grabberPosition }
        <Grabber 
          { grabberPosition }
          { resizing } 
          { state }
        />
      {/if}

    </div>
  {/if}
<style>
  .superContainer {
    flex: auto;
    position: relative;
    display: flex;
  }

  .spectrum-OpacityCheckerboard {
    block-size: unset;
    inline-size: unset;
  }
  .nested {
    --spectrum-opacity-checkerboard-square-dark: var(--random-color) !important ;
  }
  .grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    grid-column-gap: 10px;
    grid-row-gap: 10px; 
  }

  :global(.grid > .component) {
    display: inline;
  }
</style>