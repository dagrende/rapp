<template>
  <div id="app">
    <div class="events" @click="selectedEventIndex = null">
      <div class="event" v-for="(event, i) in events"
        @click="selectEvent(events.indexOf(event)); $event.stopPropagation()"
        :class="{selected: i === selectedEventIndex}">
        <div class="row1">
          <input type="text" :value="event.type" @change="changeEvent(event, 'type', $event.target.value)"
            :hidden="i != selectedEventIndex"
            @focus="$event.target.select()">
          <span :hidden="i == selectedEventIndex">{{event.type}}</span>
          <span class="filler"></span>
          <input type="text" :value="event.startTime.format('YY-MM-DD h:mm')"
            @change="changeEvent(event, 'startTime', moment($event.target.value, 'YY-MM-DD h:mm'))"
            :hidden="i != selectedEventIndex"
            @focus="$event.target.select()">
          <span :hidden="i == selectedEventIndex">{{event.startTime.format('YY-MM-DD h:mm')}}</span>
        </div>
        <div class="row2">
          <input type="text"
            :value="event.comment"
            @change="changeEvent(event, 'comment', $event.target.value)"
            :hidden="i != selectedEventIndex"
            @focus="$event.target.select()">
          <span :hidden="i == selectedEventIndex">{{event.comment}}</span>
        </div>
      </div>
      <div class="event dummy-event">
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
  import uuidv4 from 'uuid/v4';

  export default {
    name: 'app',
    data () {
      return {
        events: [
          {id: uuidv4(), type: 'bbi', startTime: moment(), comment: 'hej du glade'},
          {id: uuidv4(), type: 'hem', startTime: moment(), comment: ''},
          {id: uuidv4(), type: 'bbi', startTime: moment(), comment: 'tag en spade'}
        ],
        eventType: '',
        selectedEventIndex: null,
      }
    },
    methods: {
      selectEvent(i) {
        if (this.selectedEventIndex === i) {
          //this.selectedEventIndex = null;
        } else {
          this.selectedEventIndex = i;
        }
      },
      addEvent(eventType) {
        let newEvent = {id: uuidv4(), type: eventType, startTime: moment(), comment: ''};
        this.events.push(newEvent)
        this.selectEvent(null);
        createEvent(newEvent);
      },
      changeEvent(event, fieldName, value) {
        event[fieldName] = value;
        saveEvent(newEvent);
      },
      createEvent(event) {
        console.log('createEvent(',JSON.stringify(event),')');
      },
      saveEvent(event) {
        console.log('saveEvent(',JSON.stringify(event),')');
      },
      moment
    },
    created() {
      console.log('created');
      
    },
    computed: {
      eventTypes() {
        let typeSet = new Set();
        let types = [];
        for (let i = this.events.length - 1; i >= 0; --i) {
          let type = this.events[i].type;
          if (!typeSet.has(type)) {
            typeSet.add(type);
            types.push(type);
          }
        }
        return types
      }
    }
  }
</script>

<style>
  html, body {
    padding: 0;
    margin: 0;
  }
  #app {
    display: flex;
    flex-direction: column;
    height: calc(100vh);
    background-color: #e0e0e0;
    font-family: Arial, Helvetica, sans-serif;
  }
  .events {
    flex: 1;
    overflow: scroll;
  }
  .event {
    display: flex;
    flex-direction: column;
    padding: .5em;
    border: solid #aaa 1px;
  }
  .event.dummy-event {
    min-height: 3em;
  }
  .event .row1 {
    display: flex;
    font-size: 130%;
  }
  .event .row2 {
    display: flex;
  }
  .event  input {
    flex: 1;
    font-size: 1em;
  }
  .event.selected {
    background-color: #eeeeee;
  }
  .event-type {
  }
  .filler {
    flex: 1;
  }
  .event-time {
  }
  .event-type-filter {
    display: flex;
    padding: .5em;
    background-color: #000;
  }
  .event-type-filter input {
    flex: 1;
  }
  .event-time {

  }
  .event-types {
    flex: 1;
    overflow: scroll;
    padding: .1em;
  }
  .event-types .event-type {
    padding: .5em;
    border: solid #eee 1px;
    font-size: 130%;
  }
</style>
