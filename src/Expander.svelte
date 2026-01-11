<script>
  export let collapsed;
  export let collapseIcon;
</script>

{#if !collapsed}
  <svg
    xmlns="http://www.w3.org/2000/svg"
    width="16"
    height="16"
    viewBox="0 0 24 24"
    fill="currentColor"
    ><path
      d="M15.0712 4.92889L16.4854 6.34311L11.8286 11L21.0004 11.0001L21.0004 13.0001L11.8286 13L16.4854 17.6568L15.0712 19.071L8.00016 12L15.0712 4.92889ZM4.00037 18.9999L4.00037 4.99991H6.00037L6.00037 18.9999H4.00037Z"
    ></path></svg
  >
{:else if collapseIcon}
  <i class={collapseIcon}></i>
{:else}
  <svg
    xmlns="http://www.w3.org/2000/svg"
    width="16"
    height="16"
    viewBox="0 0 24 24"
    fill="currentColor"
  >
    <path
      d="M17.1717 11.0001L12.5148 6.34323L13.929 4.92902L21.0001 12.0001L13.929 19.0712L12.5148 17.6569L17.1716 13.0001L7.9998 13.0002L7.99978 11.0002L17.1717 11.0001ZM3.99985 19L3.99985 5.00003H5.99985V19H3.99985Z"
    >
    </path>
  </svg>
{/if}
