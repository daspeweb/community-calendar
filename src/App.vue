<template>
  <div id="root">
    <div class="modal-events-none">
      <div class="container-modal">
      </div>
    </div>
    <div class="container-main">
      <div class="check-box">
        <div class="filterDate-box">
          <input class="h-34px" type="date" v-model="dateToFilterEvents" title="Filtre por data">
        </div>
        <select class="h-34px " v-model="selectedEventStatus">
          <option v-for="option in this.eventStatusOptionArr" v-bind:key="option.value" :value="option.value">{{option.label}}</option>
        </select>
      </div>
      <div class="container-button-add-event">
      </div>
    </div>
    <div id="append-events" style="height: 100%; background: white; overflow: hidden;">
      <div v-for="year in Object.keys(eventArrEnhanced)" v-bind:key="year">
        <div v-bind:id="'year-' + year"  class="year">
          <div v-for="month in Object.keys(eventArrEnhanced[year]).sort()" v-bind:key="month">
            <div v-bind:id="'year-' + year + '-' + month"  class="month">
              <div v-for="day in (Object.keys(eventArrEnhanced[year][month]).sort())" v-bind:key="day">
                <div v-bind:id="'year-' + year + '-' + month + '-' + day" class="day">
                  <div class="day-info">
                    <span>{{ moment.utc(new Date(year, month - 1, Number(day))).format('ddd[,] DD [de] MMM [de] Y') }}</span>
                  </div>
                  <div v-for="event in eventArrEnhanced[year][month][day]" v-bind:key="event.Id">
                    <div class="container-event-info">
                      <div class="event-title">
                        <div v-on:click="navSObject(event.Id)" class="slds-icon_container slds-icon-standard-account " title="clique aqui para acessar o registro">
                          <svg aria-hidden="true" class="slds-icon slds-icon_xx-small">
                            <use v-bind:xlink:href="urlPrefix + '/action-sprite/svg/symbols.svg#preview'"></use>
                          </svg>

                        </div>
                        <div v-on:click="editSObject(event.Id)" class="slds-icon_container slds-icon-standard-account " title="clique aqui para editar o registro">
                          <svg aria-hidden="true" class="slds-icon slds-icon_xx-small">
                            <use v-bind:xlink:href="urlPrefix + '/action-sprite/svg/symbols.svg#edit'"></use>
                          </svg>
                        </div>
                      </div>
                      <div  class="event-info " v-bind:class="event.colorClass">
                        <div class="name">
                          <p class="slds-test">{{ event.Subject }} </p>
                          <small> {{ event.relative }}</small>
                        </div>
                        <div  class="durating">
                          <small><b>Horas: </b></small><br/>
                          <small><b>{{ event.startingTime }} - {{ event.endingTime }}</b></small>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

</template>


<script>


import PostMessenger from "@/components/PostMessenger"
import MessageHandler from "@/components/MessageHandler"
import PostMessengerMockHandler from "@/components/PostMessengerMockHandler"
// import Helper from "@/components/Helper";
import moment from 'moment'
import 'moment/locale/pt-br'

moment.locale('pt-br');

export default {
  name: 'Community-Calendar',
  components: {

  },
  watch: {
    selectedEventStatus: function () {
      this.filterEvents()
    },
    dateToFilterEvents: function (){
      this.filterEvents()
    }
  },
  computed: {
    eventArrEnhanced(){
      let objAux = this.filteredEventArr.map(event => {
        //calcula status da task ou event
        if (event.StartDateTime){//se for evento
          if (event.Status_Event__c === 'Concluído'){
            event.generalStatus = 'concluido'
          }else{
            let now = moment()
            let eventDueTime = moment(Date.parse(event.StartDateTime))
            if (eventDueTime < now){
              event.generalStatus = 'atrasado'
            }else{
              event.generalStatus = 'aberto'
            }
          }
        }else{//se for task
          if (event.Status === 'Concluído'){
            event.generalStatus = 'concluido'
          }else{
            let now = moment()
            let eventDueTime = moment(Date.parse(event.ActivityDate))
            if (eventDueTime < now){
              event.generalStatus = 'atrasado'
            }else{
              event.generalStatus = 'aberto'
            }
          }
        }

        let dtAux = event.StartDateTime
            ? moment(event.StartDateTime)
            : moment(event.ActivityDate).utc(false)

        event.year = dtAux.format('Y')
        event.month = dtAux.format('MM')
        event.day = dtAux.format('DD')
        event.dayOfWeek = dtAux.format('ddd')
        event.monthName = dtAux.format('MMM')

        event.startingTime = event.StartDateTime
            ? moment(Date.parse(event.StartDateTime)).format('H:ss')
            : ''

        event.startingTime = event.EndDateTime
            ? moment(Date.parse(event.EndDateTime)).format('H:ss')
            : ''

        let dateIni = event.ActivityDate;
        // let dateIniSFormmted = data.ActivityDate;
        //let initHour = '';
        if (!dateIni || 0 === dateIni.length) {
          dateIni = event.StartDateTime;
          let formated_Date = dateIni;
          let date = new Date(formated_Date) // formated_Date - SDK returned date
          let minutes = date.getMinutes()
          if(minutes == '0'){
            minutes = '00'
          }
          //initHour = `${date.getHours()}:${minutes}`;
        }
        let whoName = '';
        if(typeof event.Who !== 'undefined'){
          whoName = event.Who.Name;
        }
        event.relative = `Relativo à: ${whoName}`;
        if(event.Lead__c){
          event.relative = `Empresa: ${event.Empresa__c}`;
        }

        if (event.generalStatus === 'aberto' ) {
          event.colorClass = 'addColorProgressOpened';
        }else if (event.generalStatus === 'concluido') {
          event.colorClass = 'addColorProgressConcluded';
        } else if (event.generalStatus === 'atrasado' )  {
          event.colorClass = 'addColorProgressNotStarted';
        }
        return event
      })

      let groupBy = function(xs, key) {
        return xs.reduce(function(rv, x) {
          (rv[x[key]] = rv[x[key]] || []).push(x);
          return rv;
        }, {});
      }

      let grouped = groupBy(objAux, 'year')
      Object.keys(grouped).forEach(year => {
        grouped[year] = groupBy(grouped[year], 'month')
        Object.keys(grouped[year]).forEach(month => {
          grouped[year][month] = groupBy(grouped[year][month], 'day')
        })
      })
      console.log(JSON.parse(JSON.stringify(grouped)))
      return grouped
    }
  },
  data: function() {
    return {
      urlPrefix: document.querySelector('#fixed-resource-path').value || document.querySelector('#fixed-testing-path').value,
      moment: moment,
      selectedEventStatus: 'todos',
      eventStatusOptionArr: [
        {value: 'todos', label: 'Todos'},
        {value: 'aberto', label: 'Em aberto'},
        {value: 'atrasado', label: 'Atrasados'},
        {value: 'concluido', label: 'Concluídos'}
      ],
      eventArr: [],
      filteredEventArr: [],
      dateToFilterEvents: Date
    }
  },
  methods: {
    filterEvents(){
      console.log('filterEvents..')
      let self = this
      this.filteredEventArr = this.eventArr.filter(function(event){
        if(self.selectedEventStatus === 'todos') {return true}
        return event.generalStatus === self.selectedEventStatus
      })
      if (typeof this.dateToFilterEvents === 'string' && this.dateToFilterEvents.length > 0){
        var date = new Date(this.dateToFilterEvents)
        var userTimezoneOffset = date.getTimezoneOffset() * 60000;
        let dt = moment.utc(new Date(date.getTime() + userTimezoneOffset))
        this.filteredEventArr = this.filteredEventArr.filter(function(event){
          let dtAux = event.StartDateTime
              ? moment(event.StartDateTime)
              : moment(event.ActivityDate)
          dtAux.set({hour:0,minute:0,second:0,millisecond:0})
          return dtAux.format('DD MMM Y') === dt.format('DD MMM Y')
        })
      }

    },
    newinit: () => {
      console.log('solicitando dados pra LC...')
      PostMessenger.methods.postParent('init', {}, '*')
    },
    navSObject: (id) => {
      PostMessenger.methods.postParent('redirect', {id: id}, '*')
    },
    editSObject: (id) => {
      PostMessenger.methods.postParent('redirectEdit', {id: id}, '*')
    }
  },
  created() {
    let self = this
    PostMessengerMockHandler.methods.listenAll()//apenas para testes
    console.log('carregando VF...')

    MessageHandler.methods.register('loadEventsResponse', function (data){
      self.eventArr = JSON.parse(data.eventsJSON);
      self.filterEvents()
    })

    this.newinit();
  }


}
</script>

<style>

.container-main {
  transition-duration: .2s;
  /*padding-bottom: 2.5rem;*/
}

.month .day {
  height: max-content;
  min-height: 80px;
  display: flex;
  flex-direction: column;
}

.check-box {
  display: flex;
  justify-content: space-between;

  text-align: right;
  color: #9d9d9d;
  background: white;
  padding: .2rem;
  font-weight: bold;
  position: sticky;
  top: 0;
  z-index: 9999999;
} .check-colors {
    text-align: left;
    color: #9d9d9d;
  }

.container-main{
  height: max-content;
}

.year{
  height: calc(100% - 36px);
}

.year .month {
  height: max-content;
}

.month .day-info {
  color: white;
  background: #9BB6D5;
  padding: .2rem;
  font-weight: bold;
  position: sticky;
  top: 0px;
  display: flex;
  height: 30px;
}

.month .day-info span {
  margin-right: .25rem;
}

.month .event-info {
  max-width: 100%;
  display: flex;
  padding: .25rem;

  border-bottom: solid 1px #808080b0;
}


.month .event-title {
  max-width: 100%;
  margin-left: .25rem;
  display: flex;
  padding: .25rem;
  color: #9d9d9d;
  border-bottom: solid 0px #808080b0;
}

.addColorBlue{
  color: blue;
}
.addColorProgressOpened{
  color:#707070;
}
.addColorProgressNotStarted{
  color: #e64236;
}
.addColorProgressConcluded{
  color: #2fd14a;
}

.month .event-title a {
  padding: .25rem;
  margin-right: .25rem;
  color: #fff;

}

.month .event-title svg {
  margin: 0 auto;
  display: block;
}

.month .event-title>div {
  display: flex;
  background-color: #9BB6D5;
  align-items: center;
  align: center;
  justify-content: center;
  margin-right: .25rem;
  border-radius: 50%;
  height: 25px;
  width: 25px;

}
.month .event-title>div:hover {
  background-color: rgb(35, 118, 212);
}
.month .event-title>div:active {
  background-color: rgb(35, 118, 212);
}

.month .container-event-info .event-info .durating {
  width: 25%;
  display: block;
  height: max-content;
  margin: auto;
  text-align: center;
  border-left: 1px solid #0070D2;
}

.month .container-event-info .event-info .name {
  width: 75%;
  display: block;
  padding-left: .5rem;
}

.month .container-event-info .event-info .name p {
  margin: 0;
}


.container-modal {
  position: fixed;
  z-index: 1030;
  width: calc(100% - 1rem);
  margin: .5rem;
  border-radius: 4px;
  box-shadow: #00000069 0px 0px 7px 0px;
  background: #fff;
}

@keyframes fadeInTop {
  from {
    transform: translateY(-100%);
  }
  to {
    transform: translateY(0%);
  }
}

@keyframes fadeOutTop {
  from {
    transform: translateY(0%);
  }
  to {
    transform: translateY(-110%);
  }
}

.container-modal>div {
  padding: .5rem;
}

.container-modal .title {
  display: flex;
  justify-content: space-between;
}



.container-modal .title button {
  border: none;
  background: none;
  color: red;
  height: max-content;
  margin: 0;
  padding: 0;
}

.container-modal .event .info p {
  margin: 0;
}

.container-modal .event .info p:nth-child(1) {
  font-size: 1rem;
  margin: .25rem 0 .25rem 0;
}

.container-modal .event .info p:nth-child(2) {
  font-size: 0.75rem;
  margin: .25rem 0 .25rem 0;
}

.container-modal .event .info p:nth-child(3) {
  font-size: .7rem;
}

.container-modal form {
  display: table;
}

.container-modal form input {
  width: calc(50% - .9rem);
  padding: .15rem;
  margin: .25rem;
  height: 1.3rem;
  border-radius: 4px;
  transition-duration: .1s;
  outline-color: transparent;

}
input, select{
  border: #cccccc solid 1px !important;
}

.container-modal form input[name=play],
.container-modal form input[name=end] {
  width: calc(25% - .9rem);
}

.container-modal form input:focus {
  border-color: #0000ffb3!important;
}

.container-modal form button {
  background: #e5e5e5;
  border: none;
  padding: .25rem;
  margin-left: .25rem;
  border-radius: 2px;
  font-weight: bold;
  box-shadow: 0px 0px 3px 1px #0000004d;
}

.close {
  color: #aaaaaa;
  font-size: 18px;
  font-weight: bold;
}
.filterDate-box{
  display: inline-block;
  position: sticky;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}
[type="date"] {
  background:#fff url(https://cdn1.iconfinder.com/data/icons/cc_mono_icon_set/blacks/16x16/calendar_2.png)  97% 50% no-repeat ;
}
[type="date"]::-webkit-inner-spin-button {
  display: none;
}
[type="date"]::-webkit-calendar-picker-indicator {
  opacity: 0;
}

.h-34px{
  height: 34px;
}


</style>