<template>
  <main class="page">
    <section class="page__card">
      <header class="page__row">
        <h1 class="page__title">Календарь (тестовое)</h1>
        <div class="page__spacer"></div>
        <select v-model="lang" class="page__select">
          <option value="ru">ru</option>
          <option value="en">en</option>
        </select>
      </header>

      <CalendarLite
        :initial-date="''"
        :locale="lang"
        :week-starts-on="1"
        @select="picked = $event"
      />

      <footer class="page__row page__row--muted">
        Выбрано: <b class="page__picked">{{ picked }}</b>
      </footer>
    </section>
  </main>
</template>

<script>
import CalendarLite from './components/CalendarLite.vue';
export default {
  name: 'App',
  components: { CalendarLite },
  data: () => ({ picked: '—', lang: 'ru' })
};
</script>

<style scoped>
.page{
  min-height:100vh;
  display:grid;
  place-items:center;
  background:#f7f7f8;
  padding:16px;
}
.page__card{
  width:100%;
  max-width:480px;
  background:#fff;
  border:1px solid #e6e6e6;
  border-radius:16px;
  box-shadow:0 6px 18px rgba(0,0,0,.06);
  padding:14px;
}
.page__row{
  display:flex;
  align-items:center;
  gap:8px;
}
.page__row--muted{
  color:#64748b;
  margin-top:8px;
}
.page__title{
  font-size:16px;
  margin:0;
}
.page__spacer{
  flex:1;
}
.page__select{
  padding:6px 10px;
  border:1px solid #e5e7eb;
  border-radius:10px;
  background:#fff;
}
.page__picked{
  margin-left:6px;
}
@media (max-width:480px){
  .page__card{ max-width:360px; }
}
</style>
