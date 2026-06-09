<script>
  import { setContext } from "svelte";
  import { writable } from "svelte/store";

  let {
    labelPos,
    gridColumns,
    labelWidth,
    disabled = false,
    readonly = false,
    children,
  } = $props();

  // svelte-ignore state_referenced_locally
  const groupDisabled = writable(disabled);
  // svelte-ignore state_referenced_locally
  const groupReadonly = writable(readonly);

  // svelte-ignore state_referenced_locally
  setContext("field-group", labelPos);
  // svelte-ignore state_referenced_locally
  setContext("field-group-columns", gridColumns);
  // svelte-ignore state_referenced_locally
  setContext("field-group-label-width", labelWidth);
  setContext("field-group-disabled", groupDisabled);
  setContext("field-group-readonly", groupReadonly);

  $effect(() => {
    groupDisabled.set(disabled);
    groupReadonly.set(readonly);
  });
</script>

{@render children?.()}