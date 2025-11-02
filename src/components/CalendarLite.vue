<template>
  <div class="calendar" role="application" aria-label="Calendar">
    <div class="calendar__header">
      <div class="calendar__nav">
        <button class="calendar__btn" @click="go(-1)" aria-label="Previous month">◀</button>
        <button class="calendar__btn" @click="go(1)" aria-label="Next month">▶</button>
      </div>
      <div class="calendar__title" aria-live="polite">{{ title }}</div>
      <div class="calendar__spacer"></div>
    </div>

    <div class="calendar__week">
      <div v-for="w in weekdayLabels" :key="w" class="calendar__weekday">{{ w }}</div>
    </div>

    <div class="calendar__grid">
      <div
        v-for="c in cells"
        :key="c.iso"
        class="calendar__day"
        :class="{
          'calendar__day--outside': c.other,
          'calendar__day--today': c.today,
          'calendar__day--selected': c.active,
          'calendar__day--weekend': c.wknd
        }"
        @click="pick(c)"
      >{{ c.day }}</div>
    </div>

    <div class="calendar__legend">
      Клик по дню эмитит событие <code>@select</code> со строкой <code>YYYY-MM-DD</code>.
    </div>
  </div>
</template>

<script>
const pad = n => (n < 10 ? '0' + n : '' + n);
const toISO = (y, m, d) => `${y}-${pad(m)}-${pad(d)}`;
const parseISO = s => {
  const m = /^(\d{4})-(\d{2})-(\d{2})$/.exec(s || '');
  if (!m) return null;
  const dt = new Date(+m[1], +m[2] - 1, +m[3]);
  return isNaN(dt.getTime()) ? null : dt;
};
const addDays = (date, delta) => {
  const x = new Date(date);
  x.setDate(x.getDate() + delta);
  return x;
};
const sameDate = (a, b) =>
  a.getFullYear() === b.getFullYear() &&
  a.getMonth() === b.getMonth() &&
  a.getDate() === b.getDate();

const LOCALES = {
  ru: {
    months: ['Январь','Февраль','Март','Апрель','Май','Июнь','Июль','Август','Сентябрь','Октябрь','Ноябрь','Декабрь'],
    weekdays: ['Пн','Вт','Ср','Чт','Пт','Сб','Вс']
  },
  en: {
    months: ['January','February','March','April','May','June','July','August','September','October','November','December'],
    weekdays: ['Mon','Tue','Wed','Thu','Fri','Sat','Sun']
  }
};

export default {
  name: 'CalendarLite',
  props: {
    initialDate: { type: String, default: '' },
    locale: { type: String, default: 'ru' },
    weekStartsOn: { type: Number, default: 1 }
  },
  data() {
    const base = parseISO(this.initialDate) || new Date();
    return {
      viewYear: base.getFullYear(),
      viewMonth: base.getMonth(),
      selected: { y: base.getFullYear(), m: base.getMonth() + 1, d: base.getDate() }
    };
  },
  computed: {
    loc() { return LOCALES[this.locale] || LOCALES.ru; },
    title() { return `${this.loc.months[this.viewMonth]} ${this.viewYear}`; },
    weekdayLabels() {
      if ((this.weekStartsOn || 1) === 1) return this.loc.weekdays;
      const a = this.loc.weekdays.slice();
      a.unshift(a.pop());
      return a;
    },
    cells() {
      const first = new Date(this.viewYear, this.viewMonth, 1);
      const offset = (first.getDay() - (this.weekStartsOn || 1) + 7) % 7;
      const start = new Date(this.viewYear, this.viewMonth, 1 - offset);

      const today = new Date();
      const arr = [];
      for (let i = 0; i < 42; i++) {
        const dt = addDays(start, i);
        const y = dt.getFullYear();
        const m = dt.getMonth() + 1;
        const d = dt.getDate();
        arr.push({
          iso: toISO(y, m, d),
          other: dt.getMonth() !== this.viewMonth,
          today: sameDate(dt, today),
          active: (y === this.selected.y && m === this.selected.m && d === this.selected.d),
          wknd: [0, 6].includes(dt.getDay()),
          y,
          m,
          day: d
        });
      }
      return arr;
    }
  },
  methods: {
    go(dir) {
      let m = this.viewMonth + dir;
      let y = this.viewYear;
      if (m < 0) { m = 11; y--; }
      if (m > 11) { m = 0; y++; }
      this.viewMonth = m;
      this.viewYear = y;
    },
    pick(c) {
      this.selected = { y: c.y, m: c.m, d: c.day };
      this.viewYear = c.y;
      this.viewMonth = c.m - 1;
      this.$emit('select', toISO(c.y, c.m, c.day));
    }
  },
  watch: {
    initialDate(val) {
      const dt = parseISO(val);
      if (!dt) return;
      this.viewYear = dt.getFullYear();
      this.viewMonth = dt.getMonth();
      this.selected = { y: dt.getFullYear(), m: dt.getMonth() + 1, d: dt.getDate() };
    }
  }
};
</script>

<style scoped>
.calendar{
  width:100%;
  max-width:480px;
  margin-top:12px;
  background:var(--c-bg);
  border:1px solid var(--c-brd);
  border-radius:var(--r);
  box-shadow:var(--shadow);
  color:var(--c-ink);
}
.calendar__header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  padding:12px 14px;
  border-bottom:1px solid var(--c-brd);
}
.calendar__nav{
  display:flex;
  gap:8px;
}
.calendar__btn{
  width:36px;
  height:36px;
  display:grid;
  place-items:center;
  padding:0;
  border-radius:12px;
  background:#fff;
  border:1px solid var(--c-brd);
  transition:background .15s,border-color .15s,transform .06s ease;
  cursor:pointer;
}
.calendar__btn:hover{
  background:#F6FAFF;
  border-color:#D0D5DD;
}
.calendar__btn:active{
  transform:translateY(1px);
}
.calendar__title{
  font-weight:700;
  font-size:16px;
  letter-spacing:.2px;
}
.calendar__spacer{
  width:74px;
}
.calendar__week,
.calendar__grid{
  display:grid;
  grid-template-columns:repeat(7,minmax(0,1fr));
}
.calendar__week{
  padding:8px 10px 0;
  color:#667085;
  font-weight:600;
  font-size:12px;
}
.calendar__weekday{
  justify-self:center;
}
.calendar__grid{
  gap:10px;
  padding:12px;
}
.calendar__day{
  aspect-ratio:1/1;
  min-height:42px;
  display:grid;
  place-items:center;
  border-radius:12px;
  border:1px solid transparent;
  cursor:pointer;
  user-select:none;
  font-weight:600;
  transition:background .12s,border-color .12s,box-shadow .12s,color .12s;
}
.calendar__day:hover{
  background:var(--c-accent-hover);
  border-color:#D0D5DD;
}
.calendar__day--outside{
  color:var(--c-muted);
}
.calendar__day--today{
  box-shadow:inset 0 0 0 2px var(--c-today);
  border-color:transparent;
}
.calendar__day--selected{
  background:var(--c-accent-soft);
  color:var(--c-ink);
  border-color:transparent;
  box-shadow:inset 0 0 0 2px var(--c-accent);
}
.calendar__legend{
  padding:10px 12px;
  border-top:1px solid var(--c-brd);
  color:#667085;
  font-size:12px;
}
@media (max-width:480px){
  .calendar{
    max-width:360px;
  }
  .calendar__grid{
    gap:8px;
    padding:10px;
  }
  .calendar__day{
    min-height:38px;
    border-radius:10px;
  }
  .calendar__title{
    font-size:15px;
  }
}
</style>
