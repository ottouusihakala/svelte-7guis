<script lang="ts">
  import { formatISO, format, parseISO, addDays } from 'date-fns';
  import DateInput from './DateInput';

	const ONE_WAY: string = 'ONE_WAY';
	const TWO_WAY: string = 'TWO_WAY';

	const flightOptions: { value: string, label: string }[] = [
		{ value: ONE_WAY, label: 'one-way flight' },
    { value: TWO_WAY, label: 'return flight' }
	];

  const formatISODate = (date: Date) => formatISO(date, { representation: 'date' });
  const minReturnDate = (date: string) => formatISODate(addDays(parseISO(date), 1));

  let flight: string;
  let departureDate: string = formatISODate(new Date());
  let returnDate: string;

  const formOnSubmit = (e: Event & { currentTarget: HTMLFormElement | null }) => {
    e.preventDefault();
    console.log('form is valid ', e.currentTarget);
  };
</script>

<main>
	<h1>Flight Booker</h1>
  <section>
    <form on:submit={formOnSubmit}>
      <fieldset>
        <label>
          Flight direction
          <select bind:value={flight}>
            {#each flightOptions as opt}
              <option value={opt.value}>{opt.label}</option>
            {/each}
          </select>
        </label>
        <DateInput label="Departure" date={departureDate} />
        <label>
          Return
          <input 
            on:blur={(e) => e.currentTarget.reportValidity()} 
            min={minReturnDate(departureDate)} 
            bind:value={returnDate} 
            type="date" 
            disabled={flight !== TWO_WAY} 
            required={flight === TWO_WAY}
          />
        </label>
      </fieldset>
      <input type="submit" value="Book" />
    </form>
  </section>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

  form {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  fieldset {
    border: none;
    width: fit-content;
  }

  label {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
  }

  label > input {
    margin-left: 1em;
  }

  input[type="submit"] {
    padding: 1em 2em;
  }

  input:invalid {
    border-color: red;
    border-width: 2px;
  }
</style>