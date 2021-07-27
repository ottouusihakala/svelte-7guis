<script lang="ts">
  import { afterUpdate } from 'svelte';
  import { 
    getDaysInMonth, 
    getMonth, 
    getYear, 
    getDate, 
    getISODay,
    parseISO,
    formatISO, 
    format, 
    isSameDay, 
    isSameWeek,
    endOfMonth,
    isSameMonth
  } from 'date-fns';
  import { enUS, enGB, fi } from 'date-fns/locale';
  import { groupWith } from 'ramda';   
  import Icon from 'svelte-awesome';
  import { chevronLeft, chevronRight, remove } from 'svelte-awesome/icons';

  const LANG_FI: string = 'fi';
  const LANG_EN_US: string = 'enUS';
  const LANG_EN_UK: string = 'enUK';
  
  export let date: Date = new Date();
  export let setDate: (value: Date) => void = (value) => {
    date = value;
  };
  export let onClose: () => void = () => {};
  let currentMonth: number = getMonth(date);
  let currentYear: number = getYear(date);
  let currentWeeks: Date[][] = [];
  export let lang: string = 'enUS';
  const getLocale = (lang: string) => {
    switch (lang) {
      case LANG_FI:
        return fi;
      case LANG_EN_US:
        return enUS;
      case LANG_EN_UK:
        return enGB;
      default: 
        return enUS;
    }
  };
  let locale: Locale = getLocale(lang);

  const weekStartsWithSunday = lang === LANG_EN_US;

  type GroupWithDateFn = (list: readonly Date[]) => Date[][];

  const groupWithSameWeek: GroupWithDateFn = 
    groupWith((date1: Date, date2: Date) => isSameWeek(date1, date2, { locale }));

  type GetMonthWeeksOptions = { weekStartsWithSunday: boolean, orgFn: GroupWithDateFn };

  const defaultGetMonthWeeksOptions: GetMonthWeeksOptions = {
    weekStartsWithSunday: false,
    orgFn: groupWithSameWeek
  };

  const getMonthWeeks: (year: number, month: number, options?: GetMonthWeeksOptions) => Date[][] = 
      (year: number, month: number, options: GetMonthWeeksOptions = defaultGetMonthWeeksOptions) => {
    const { weekStartsWithSunday, orgFn } = options;
    const firstDayOfCurrentMonth = new Date(year, month, 1);
    const daysInMonth = getDaysInMonth(firstDayOfCurrentMonth);
    const daysBeforeMonth = getISODay(firstDayOfCurrentMonth) - (weekStartsWithSunday ? 0 : 1);
    const dayIndicesPreceeding = [...Array(daysBeforeMonth).keys()].map((i) => i - daysBeforeMonth);
    const lastDayOfMonth = endOfMonth(firstDayOfCurrentMonth);
    const lastDifference = 7 - (getISODay(lastDayOfMonth) + (weekStartsWithSunday ? 1 : 0));
    const daysAfterMonth = [...Array(lastDifference).keys()].map((i) => lastDayOfMonth.getDate() + i);
    const dayNumbers = [...dayIndicesPreceeding, ...Array(daysInMonth).keys(), ...daysAfterMonth].map((i) => i + 1);
    const days = dayNumbers.map((i) => new Date(year, month, i));
    const daysGroupedWithWeek = orgFn(days);
    return daysGroupedWithWeek;
  };
  
  const weekDayIndices = weekStartsWithSunday ? [...Array(7).keys()] : [...[...Array(6).keys()].map((i) => i + 1), 0];

  const onClickDate = (e: Event & { currentTarget: HTMLButtonElement }) => {
    setDate(parseISO(e.currentTarget.value));
  };

  const onClickNextMonth = () => {
    if (currentMonth === 11) {
      currentMonth = 0;
      currentYear = currentYear + 1;
    } else {
      currentMonth = currentMonth + 1;
    }
  };

  const onClickPrevMonth = () => {
    if (currentMonth === 0) {
      currentMonth = 11;
      currentYear = currentYear - 1;
    } else {
      currentMonth = currentMonth - 1;
    }
  };

  afterUpdate(() => {
    currentWeeks = getMonthWeeks(currentYear, currentMonth, { weekStartsWithSunday, orgFn: groupWithSameWeek });
  });
</script>

<div class="datePickerWrapper">
  <div class="datePickerPanel">
    <div class="topRow">
      <div class="currentDate">{format(date, fi.formatLong?.date(), { locale })}</div>
      <button on:click={onClose} type="button">
        <Icon data={remove} /> 
      </button>
    </div>
    <div class="monthSelector">
      <button on:click={onClickPrevMonth} type="button">
        <Icon data={chevronLeft} />
      </button>
      <span class="currentMonth">{locale.localize?.month(currentMonth)} {currentYear}</span>
      <button on:click={onClickNextMonth} type="button">
        <Icon data={chevronRight} />
      </button>
    </div>
    <div class="weekDayLabels">
      {#each weekDayIndices as weekDayIndex}
        <div>{locale.localize?.day(weekDayIndex).substring(0, 2)}</div>
      {/each}
    </div>
    <div class="dateSelector">
      {#each currentWeeks as week}
        <div class="week">
          {#each week as day}
            <button 
              type="button" 
              class={`day ${isSameDay(date, day) ? 'selectedDate' : ''}`} 
              disabled={!isSameMonth(day, new Date(currentYear, currentMonth, 1))}
              value={formatISO(day)}
              on:click={onClickDate}
            >
              {getDate(day)}
            </button>
          {/each}
        </div>
      {/each}
    </div>
  </div>
</div>

<style>
  .datePickerWrapper {
    position: relative;
  }

  .datePickerPanel {
    position: absolute;
    border: 1px solid gray;
    background-color: white;
  }

  .currentMonth {
    font-weight: bold;
  }

  .weekDayLabels {
    padding: 0 0.5em;
    display: inline-block;
  }

  .weekDayLabels > * {
    width: 2em;
    display: inline-block;
  }

  .week {
    display: flex;
    justify-content: space-between;
    padding: 0 0.5em;
  }

  .day {
    width: 2em;
    padding: none;
  }

  .selectedDate {
    background-color: blueviolet;
    color: white;
  }
</style>