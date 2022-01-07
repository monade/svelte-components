<script lang="ts">
  import moment from 'moment';
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  export let value: string = null; //initial date value in the format: "YYYY-MM-DD"
  export let dateFormat: string = 'YYYY-MM-DD'; //format of the returned value dispatched
  export let minDate: Date = moment('1900-01-01').toDate();
  export let maxDate: Date = new Date();
  export let required: boolean = false;

  let currentDate = value ? moment(value) : moment(maxDate);
  let day: number | null = currentDate?.date() || null;
  let month: number | null = currentDate?.month() || null;
  let year: number | null = currentDate?.year() || null;

  $: years = getYears(maxDate, minDate);
  $: months = getMonths(year, maxDate);
  $: days = getDays(month, year, maxDate);

  const getYears = (maxDate, minDate) => {
    const minDate_moment = moment(minDate);
    const interval = moment(maxDate).diff(minDate, 'years', false);
    return new Array(interval + 1)
      .fill(undefined)
      .map((_, index) => minDate_moment.year() + index)
      .map(e => {
        return { id: e, label: e };
      });
  };

  const getMonths = (year, maxDate) => {
    const months = moment.months().map((e, index) => {
      return { label: e, id: index };
    });
    if (year === maxDate.getFullYear()) {
      return months.filter(e => e.id <= maxDate.getMonth());
    }
    return months;
  };

  const getDays = (month, year, maxDate) => {
    let count = 31;
    if (year === null) {
      count = moment(new Date(2016, month, 1)).daysInMonth(); //2016 -> because is a leap (bisestile) year
    } else {
      count = moment(new Date(year, month, 1)).daysInMonth();
    }
    const days = new Array(count).fill(undefined).map((_, index) => {
      return { id: index + 1, label: index + 1 };
    });
    if (year === maxDate.getFullYear() && month === maxDate.getMonth()) {
      const maxDate_moment = moment(maxDate);
      return days.filter(e => e.id <= maxDate_moment.date());
    }
    return days;
  };

  $: day && onDayChanged();
  const onDayChanged = () => {
    updateValue();
  };

  $: month && onMonthChanged();
  const onMonthChanged = () => {
    validateDay();
    updateValue();
  };

  $: year && onYearChanged();
  const onYearChanged = () => {
    validateDay();
    updateValue();
  };

  const validateDay = () => {
    if (!days.find(e => e.id === day)) {
      day = null;
    }
    if (!months.find(e => e.id === month)) {
      month = null;
    }
  };

  const updateValue = () => {
    if (year === null) {
      return;
    }
    if (month === null) {
      return;
    }
    if (day === null) {
      if (currentDate) {
        dispatch('input', null);
      }
      return;
    }
    currentDate = moment({ day: day, month: month, year: year });
    dispatch('input', currentDate.format(dateFormat));
  };
</script>

<div class="v-split-date-select form-group d-flex">
  <select {required} bind:value={day} class="form-control {day === null ? 'select-placeholder' : ''}">
    <option value={null}>Giorno</option>
    {#each days as day (day.id)}
      <option value={day.id}>{day.label}</option>
    {/each}
  </select>
  <select {required} bind:value={month} class="form-control {month === null ? 'select-placeholder' : ''}">
    <option value={null}>Mese</option>
    {#each months as month (month.id)}
      <option value={month.id}>{month.label}</option>
    {/each}
  </select>
  <select {required} bind:value={year} class="form-control {year === null ? 'select-placeholder' : ''}">
    <option value={null}>Anno</option>
    {#each years as year (year.id)}
      <option value={year.id}>{year.label}</option>
    {/each}
  </select>
</div>
