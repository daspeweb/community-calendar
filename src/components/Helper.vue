<script>
export default {
name: "Helper",
  methods: {
    splitDate(dateValue){
      let arrValue = [];
      dateValue.substring(0, 10).split('-').map(value => arrValue.push(value));

      //week data
      let weekday = [];
      weekday[0] = { prev: "Dom", full: "Domingo" };
      weekday[1] = { prev: "Seg", full: "Segunda-feira" };
      weekday[2] = { prev: "Ter", full: "Terça-feira" };
      weekday[3] = { prev: "Qua", full: "Quarta-feira" };
      weekday[4] = { prev: "Qui", full: "Quinta-feira" };
      weekday[5] = { prev: "Sex", full: "Sexta-feira" };
      weekday[6] = { prev: "Sab", full: "Sábado" };

      //month data
      let month = [];
      month[0] = { prev: "Jan", full: "Janeiro" };
      month[1] = { prev: "Fev", full: "Fevereiro" };
      month[2] = { prev: "Mar", full: "Março" };
      month[3] = { prev: "Abr", full: "Abril" };
      month[4] = { prev: "Mai", full: "Maio" };
      month[5] = { prev: "Jun", full: "Junho" };
      month[6] = { prev: "Jul", full: "Julho" };
      month[7] = { prev: "Ago", full: "Agosto" };
      month[8] = { prev: "Set", full: "Setembro" };
      month[9] = { prev: "Out", full: "Outubro" };
      month[10] = { prev: "Nov", full: "Novembro" };
      month[11] = { prev: "Dez", full: "Dezembro" };

      //console.log('arrValueEditable'+JSON.stringify(arrValue))
      //const sortedActivities = arrValue.slice().sort((a, b) => b.date - a.date)
      //console.log('arrValueEditado'+sortedActivities)
      let date = new Date(`${arrValue[1]}/${arrValue[2]}/${arrValue[0]}`);

      return {
        dayWeek: weekday[date.getDay()],
        dayMonth: date.getDate(),
        month: month[date.getMonth()],
        year: date.getFullYear()
      };
    }
  }
}
</script>
