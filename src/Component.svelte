<script>
  import { getContext, onDestroy, onMount, setContext } from "svelte"
  import "@spectrum-css/opacitycheckerboard/dist/index-vars.css"
  import LayoutPreview from "./LayoutPreview.svelte";
  import fsm from "svelte-fsm"
  import { v4 as uuidv4 } from "uuid";

  const { styleable, builderStore, Provider } = getContext("sdk")
  const component = getContext("component")
  const parentState = getContext("superLayoutManager")
   
  export let dataprovider
  export let bound
  export let flex
  export let mode
  export let direction
  export let hAlign
  export let vAlign
  export let gap
  export let wrap
  export let onClick
  export let tabsDirection
  export let splitViewDirection
  export let sections
  export let resizable
  export let collapsible

  export let title, icon, color 

  let containers = []

  $: randomColor = Math.floor(Math.random()*16777215).toString(16) + "50";
  $: nested = component ? $component.ancestors[$component.ancestors.length - 2] == "plugin/bb-component-SuperContainer" : false

  let managedFlexGrow
  let managedFlexShrink
  let width, height
  let grabberPosition
  let selectedTab 
  let repeatedItem

  const id = uuidv4();

  const state = fsm ( "container" , {
    "*" : {
      registerContainer( id, state, title, icon, color ) { 
        containers = [ ...containers, { id : id, state: state, title: title, icon: icon, color: color  } ] 
        if ( mode == "tabs" ) {
          state.setGrow()
          state.hide()
        }
        this.refresh.debounce(100);
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
          containers.splice(index,1)
          containers = containers
          this.refresh.debounce(10);
        }
      },
      setMode ( mode ) {
        this.setResizingGrabbers();
        return mode 
      },
      refresh() { 
        if ( containers.length > 0 )
          this.selectTab(containers[0].id)
        if ( bound ) 
          containers.forEach ( ({state}) => state.setRepeatedItem() )
      },
      setGrow () { managedFlexGrow = "1" },
      setShrink () { managedFlexShrink = "0"},
      setFixedWidth () {},
      setFixedHeight () {},
      setResizing ( pos ) { grabberPosition = pos; },
      setRepeatedItem () { repeatedItem = true },
      hide() { return "hidden" },
      show() { return mode },
      printName() { console.log($component.name, $state , $parentState , containers ) }
    },
    "hidden" : { },
    "disabled" : {},
    "container" : { 
      _enter() { containers.forEach( ( { state } ) => { 
        state.show() 
        if ( bound ) state.setRepeatedItem();
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
      "flex-grow": managedFlexGrow ? managedFlexGrow : flex == "grow" ? "1" : "0",
      "flex-shrink": managedFlexShrink ? managedFlexShrink : flex == "shrink" ? "1" : "0",
      "display": "flex",
      "flex-direction": direction,
      "gap": gap,
      "flex-wrap" : wrap ? "wrap" : "nowrap",
      "align-content": wrap ? direction == "row" ? vAlign : hAlign : null,
      "justify-content": direction == "row" ? hAlign : vAlign,
      "align-items": direction == "row" ? vAlign : hAlign,
      "width" : width ?? $component.styles.normal.width ?? "auto",
      "height": height ?? $component.styles.normal.height ?? "auto",
      "overflow" : "auto",
      "--random-color" : "#" + randomColor,
      "--spectrum-opacity-checkerboard-square-dark": "var(--spectrum-global-color-gray-50)",
      "--spectrum-opacity-checkerboard-square-light": bound ? "var(--spectrum-global-color-gray-100)" : "var(--spectrum-global-color-gray-75)",
      "--spectrum-opacity-checkerboard-square-size": "8px",
      "--spectrum-opacity-checkerboard-position": "left top"
    }
   }

  $: inPreview = containers?.length < 1

  onMount(() => {
    if ( parentState && nested ) parentState.registerContainer( id, state, title, icon, color )
  })

  onDestroy(() =>{ 
    if ( parentState && nested ) parentState.unregisterContainer( id )
  })

  setContext( "superLayoutManager", state )
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
{#if $state != "hidden"}
<div 
  class="superContainer"
  class:nested={$builderStore.inBuilder && nested}
  class:spectrum-OpacityCheckerboard={$builderStore.inBuilder}
  class:grid={mode == "grid" }
  class:vertical={ ( mode == "tabs" && tabsDirection != "vertical" ) 
                    || ( mode == "flexbox" && direction == "vertical" ) 
                    || ( mode == "splitview" && splitViewDirection == "vertical" ) 
                    || ( inPreview )
                  } 
  use:styleable={$component.styles}
  >

  {#if grabberPosition }
    <div 
      class="grabber" 
      class:top={grabberPosition == "top"}
      class:right={grabberPosition == "right"}
      class:bottom={grabberPosition == "bottom"}
      class:left={grabberPosition == "left"}
      />
  {/if}

  {#if containers?.length > 0}
    {#if mode == "tabs"}
      <div class="tabButtons" class:tabsVertical = { tabsDirection == "vertical" } >
        {#each containers as container, idx (container.id) }
          <div 
          class="tab"
          class:selectedTab={ container.id == selectedTab}
          class:tabVertical={tabsDirection == "vertical"}
          on:click={() => state.selectTab(container.id)}
          > { container.title || "Tab " + idx } </div>
        {/each}
      </div>
    {/if}
  {/if}

  {#if $state != "hidden" && bound && dataprovider }
    {#each dataprovider.rows as row (row._id) }
      <Provider data={row}>
        <slot />
      </Provider>
    {/each}
  {:else}
    {#if containers.length == 0}
      <LayoutPreview {mode} />
    {/if}
    {#if repeatedItem}
    <div class="repeater small" > 
      <svg xmlns="http://www.w3.org/2000/svg" height="10" viewBox="0 0 18 18" width="10">
        <defs>
          <style>
            .fill {
              fill: #464646;
            }
          </style>
        </defs>
        <title>S Refresh 18 N</title>
        <rect id="Canvas" fill="#ff13dc" opacity="0" width="18" height="18" /><path class="fill" d="M16.337,10H15.39a.6075.6075,0,0,0-.581.469A5.7235,5.7235,0,0,1,5.25,13.006l-.346-.3465L6.8815,10.682A.392.392,0,0,0,7,10.4a.4.4,0,0,0-.377-.4H1.25a.25.25,0,0,0-.25.25v5.375A.4.4,0,0,0,1.4,16a.3905.3905,0,0,0,.28-.118l1.8085-1.8085.178.1785a8.09048,8.09048,0,0,0,3.642,2.1655,7.715,7.715,0,0,0,9.4379-5.47434q.04733-.178.0861-.35816A.5.5,0,0,0,16.337,10Z" />
        <path class="fill" d="M16.6,2a.3905.3905,0,0,0-.28.118L14.5095,3.9265l-.178-.1765a8.09048,8.09048,0,0,0-3.642-2.1655A7.715,7.715,0,0,0,1.25269,7.06072q-.04677.17612-.08519.35428A.5.5,0,0,0,1.663,8H2.61a.6075.6075,0,0,0,.581-.469A5.7235,5.7235,0,0,1,12.75,4.994l.346.3465L11.1185,7.318A.392.392,0,0,0,11,7.6a.4.4,0,0,0,.377.4H16.75A.25.25,0,0,0,17,7.75V2.377A.4.4,0,0,0,16.6,2Z" />
      </svg>
    </div>
    {/if}
    <slot />
  {/if}

</div>
{/if}

<style>
  .superContainer {
    position: relative;
    display: flex;
    align-items: stretch;
    justify-content: stretch;
  }
  .nested {
    --spectrum-opacity-checkerboard-square-dark: var(--random-color) !important ;
  }
  .repeater {
    position: absolute;
    top: 16px;
    right: 16px;
    width: 36px;
    height: 36px;
    background-color: lime;
    border-radius: 50%;
    display: grid;
    align-items: center;
    justify-items: center;
    z-index: 100;
    --spectrum-opacity-checkerboard-square-dark: lime !important;
  }

  .small {
    left: 18px;
    width: 16px;
    height: 16px;
    top: calc(50% - 8px);
  }

  .grabber {
    position: absolute;
    background-color: lime;
    --grabber-thickness: 8px;
  }
  .bottom {
    bottom: 0px;
    left: 0px;
    height: var(--grabber-thickness);
    width: 100%;
  }
  .right {
    top: 0px;
    right: 0px;
    width: var(--grabber-thickness);
    height: 100%;
    z-index: 2;
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
  .tabButtons {
    display: flex;
    align-items: center;
    overflow-y: auto;
  }

  .tabsVertical {
    flex-direction: column;
    align-items: flex-end;
  }
  .tab {
    padding: 0.85rem;
    min-width: 5rem;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: var(--spectrum-global-color-gray-100);
  }

  .selectedTab {
    background-color: var(--spectrum-global-color-gray-200);
  }

  .tabVertical {
    min-width: 10rem;
    max-width: 15rem;
  }
  .tab:hover {
    cursor: pointer;
    color: var(--primaryColor);
    background-color: var(--spectrum-global-color-gray-200);
  }
  .spectrum-OpacityCheckerboard {
    block-size: unset;
  }
  .vertical {
    flex-direction: column;
  }

</style>