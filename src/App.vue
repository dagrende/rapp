<template>
  <div id="app">
    <div class="events" @click="selectedEventIndex = null">
      <div class="event" v-for="(event, i) in events"
        @click="selectEvent(events.indexOf(event)); $event.stopPropagation()"
        :class="{selected: i === selectedEventIndex}">
        <div class="row1">
          <span class="event-type">{{event.type}}</span>
          <span class="filler"></span>
          <span class="event-time">{{event.startTime.format('YY-MM-DD h:mm')}}</span>
        </div>
        <div class="row2">
          <input type="text" v-model="event.comment" :hidden="i != selectedEventIndex"
            @click="$event.target.select()">
          <span :hidden="i == selectedEventIndex">{{event.comment}}</span>
        </div>
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
        this.events.push({type: eventType, startTime: moment(), comment: ''})
        this.selectEvent(this.events.length - 1);
      }
    },
    computed: {
      eventTypes() {
        let typeSet = new Set();
        let types = [];
        for (let i = this.events.length - 1; i >= 0; --i) {
          let type = this.events[i].type;
          console.log('t', type, 1);
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
  .event .row1 {
    display: flex;
    font-size: 130%;
  }
  .event .row2 {
    display: flex;
  }
  .event .row2 input {
    flex: 1;
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
  }
</style>
