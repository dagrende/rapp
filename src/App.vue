<template>
  <div id="app">
    <div class="events">
      <div class="event" v-for="event in events">
        <span class="event-type">{{event.type}}</span><span class="event-time">{{event.startTime.format('YY-MM-DD h:mm')}}</span>
      </div>
    </div>
    <div class="event-type-filter">
      <input type="text" v-model="eventType">
      <button type="button" @click="addEvent(eventType)":disabled="eventType.trim() === ''">Start</button>
    </div>
    <div class="event-types">
      <div class="event-type" v-for="eventType in eventTypes" @click="addEvent(eventType)">
        {{eventType}}
      </div>
    </div>
  </div>
</template>

<script>
  import moment from 'moment';

  export default {
    name: 'app',
    data () {
      return {
        events: [
          {type: 'bbi', startTime: moment(), comment: 'hej du glade'},
          {type: 'hem', startTime: moment(), comment: ''},
          {type: 'bbi', startTime: moment(), comment: 'tag en spade'}
        ],
        eventType: ''
      }
    },
    methods: {
      addEvent(eventType) {
        this.events.push({type: eventType, startTime: moment(), comment: 'hej du glade'})
      }
    },
    computed: {
      eventTypes() {
        return ['bbi', 'hem']
      }
    }
  }
</script>

<style>
  #app {
    display: flex;
    flex-direction: column;
  }
  .events {
    flex: 1;
  }
  .event {
    display: flex;
    flex-direction: row;
  }
  .event-type {
    min-width: 5em;
  }
  .event-time {

  }
  .event-types {
    flex: 1;
  }
</style>
