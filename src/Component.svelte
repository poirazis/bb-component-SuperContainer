<script>
  import { getContext, onDestroy, onMount, setContext } from "svelte"
  import "@spectrum-css/opacitycheckerboard/dist/index-vars.css"
  import RepeaterPreview from "./RepeaterPreview.svelte";
  import TabControl from "./TabControl.svelte";
  import Grabber from "./Grabber.svelte";

  import fsm from "svelte-fsm"
  import { v4 as uuidv4 } from "uuid";

  const { styleable, builderStore, Provider } = getContext("sdk")
  const component = getContext("component")
  const parentState = getContext("superLayoutManager")
   
  export let dataprovider
  export let bound
  export let flex
  export let flexFactor = 1
  export let mode
  export let direction
  export let hAlign
  export let vAlign
  export let gap
  export let wrap
  
  export let tabsSize
  export let tabsAlignment
  export let tabsQuiet
  export let splitViewDirection

  export let title, icon, color 

  export let onClick

  let containers = []

  $: randomColor = bound ? "32CD3230" : Math.floor(Math.random()*16777215).toString(16) + "50";
  $: nested = component ? $component.ancestors[$component.ancestors.length - 2] == "plugin/bb-component-SuperContainer" : false

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
  let selectedTab 

  const id = uuidv4();

  const state = fsm ( "container" , {
    "*" : {
      registerContainer( id, state, title, icon, color ) { 
        containers = [ ...containers, { id : id, state: state, title: title, icon: icon, color: color  } ] 
        if ( mode == "tabs" ) {
          state.setGrow()
          state.hide()
        }
      },
      updateContainer ( id , state, title, icon , color ) {
        let index = containers.findIndex( (e) => e.id == id )
        if ( index > -1 ) {
          containers[index].state = state
          containers[index].title = title
          containers[index].icon = icon
          containers[index].color = color
        }
        this.refresh.debounce(100);
      },
      unregisterContainer( id ) { 
        let index = containers.findIndex( (e) => e.id == id )
        if ( index > -1 ) {
          containers.splice(index,1)
          containers = containers
          this.refresh.debounce(10);
        }
      },
      setMode ( mode ) {
        this.setResizingGrabbers();
        this.refresh.debounce(20);
        return mode 
      },
      refresh() { 
        if ( containers.length > 0 )
          this.selectTab(containers[0].id)
      },
      setGrow () { managedFlexGrow = "1" },
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
      show() { return mode },
      printName() { console.log($component.name, $state , $parentState , containers ) }
    },
    "hidden" : { },
    "disabled" : {},
    "container" : { 
      _enter() { containers.forEach( ( { state } ) => { 
        state.show() 
      }) 
      }
    },
    "grid" : { },
    "splitview" : {
      _enter() {
        this.setResizingGrabbers.debounce( 100 )
      },
      setResizingGrabbers() {         
        containers.forEach( ( { state } , idx ) => { 
          state.show() 
          state.setGrow()
          if ( direction == "row" && idx < containers.length - 1) 
            state.setResizing ( "right" )
          else if ( direction == "column" && idx < containers.length - 1) 
            state.setResizing ( "bottom" )
        }) },
      _exit() {
        containers.forEach( ( { state } ) => { state.setResizing( null ) }) 
      }
    },
    "accordion" : {

    },
    "tabs" : {
      refreshIndicator () { this.refresh() },
      selectTab ( tabId ) { 
        if ( tabId != selectedTab ) 
          containers.forEach( ( { id, state } ) => { 
            if ( id == tabId ) {
              state.show()
              selectedTab = tabId
            }
            else {
              state.hide()
            }
          })
      },
    }
  })

  $: state.setMode( mode, direction, bound )
  $: if ( parentState ) parentState.updateContainer( id, state, title, icon, color )

  $: $component.styles = { 
    ...$component.styles,
    normal: { 
      ...$component.styles.normal,
      "flex-grow": managedFlexGrow ? managedFlexGrow : flex == "grow" ? flexFactor : "0",
      "flex-shrink": managedFlexShrink ? managedFlexShrink : flex == "shrink" ? "1" : "0",
      "display": "flex",
      "flex-direction": mode == "tabs" ? direction == "column" ? "row" : "column" : direction,
      "gap": mode != "splitview" ? gap : "0",
      "flex-wrap" : wrap ? "wrap" : "nowrap",
      "align-content": wrap ? direction == "row" ? vAlign : hAlign : null,
      "justify-content": mode == "container" ? direction == "row" ? hAlign : vAlign : "stretch",
      "align-items": mode == "container" ? direction == "row" ? vAlign : hAlign : "stretch",
      "min-width" : width ?? $component.styles.normal.width ?? "auto",
      "max-width" : width ?? $component.styles.normal.width ?? "auto",
      "min-height": height ?? $component.styles.normal.height ?? "auto",
      "max-height": height ?? $component.styles.normal.height ?? "auto",
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
    if ( parentState && nested ) parentState.registerContainer( id, state, title, icon, color )
  })

  onDestroy(() =>{ 
    if ( parentState && nested ) parentState.unregisterContainer( id )
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
    class:vertical={ ( mode == "tabs" && direction == "column" ) 
                      || ( mode == "flexbox" && direction == "vertical" ) 
                      || ( mode == "splitview" && splitViewDirection == "vertical" ) 
                    } 
    use:styleable={$component.styles}
    >

    {#if bound && dataprovider }
      <RepeaterPreview 
      empty = { $component.children == 0 } 
      inBuilder={$builderStore.inBuilder} 
      {mode}
      />
    {/if}

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

    {#key bound}
      {#if bound && dataprovider && $component.children != 0 }
        {#each dataprovider.rows as row (row._id) }
          <Provider data={row}>
            <slot />
          </Provider>
        {/each}
      {:else}
        <slot />
      {/if}
    {/key}

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
    position: relative;
    display: flex;
  }

  .spectrum-OpacityCheckerboard {
    block-size: unset;
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
  .vertical {
    flex-direction: column;
  }
</style>