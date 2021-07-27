<script lang="ts"> 
  import { formatISO, isEqual, parseISO } from 'date-fns';
  import { afterUpdate } from 'svelte';
  import Calendar from './Calendar.svelte';

  export let date: string = formatISO(new Date(), { representation: 'date' });
  export let label: string;
  let textInputDate: string = date;
  export let required: boolean = false;
  let showCalendar: boolean = false;

  const handleOnClick = () => {
    showCalendar = true;
  };

  const setDate = (value: Date) => {
    date = formatISO(value, { representation: 'date' });
    textInputDate = date;
  };

  const handleOnKeyUp = (e: KeyboardEvent & { currentTarget: HTMLInputElement }) => {
    setDate(parseISO(e.currentTarget.value))
  };

  const onClose = () => {
    showCalendar = false;
  }
</script>

<label>
  {label}
  <input on:click={handleOnClick} on:keyup={handleOnKeyUp} required={required} type="text" bind:value={textInputDate} />
  {#if showCalendar === true}
    <Calendar date={parseISO(date)} setDate={setDate} onClose={onClose} lang="enUK" />
  {/if}
</label>
