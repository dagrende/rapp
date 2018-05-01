<template>
  <div id="app">
    <div class="events" @click="selectedEventIndex = null">
      <div class="">
        <button type="button" @click="handleAuthClick" :disabled="isSignedIn">login</button>
        <button type="button" @click="handleSignoutClick" :disabled="!isSignedIn">logout</button>
      </div>
      <div class="event" v-for="(event, i) in events"
        @click="selectEvent(events.indexOf(event)); $event.stopPropagation()"
        :class="{selected: i === selectedEventIndex}">
        <div class="row1">
          <input type="text" :value="event.type" @change="changeEvent(event, 'type', $event.target.value)"
            :hidden="i != selectedEventIndex"
            @focus="$event.target.select()">
          <span :hidden="i == selectedEventIndex">{{event.type}}</span>
          <span class="filler"></span>
          <input type="text" :value="event.startTime.format('YY-MM-DD HH:mm')"
            @change="changeEvent(event, 'startTime', moment($event.target.value, 'YY-MM-DD HH:mm'))"
            :hidden="i != selectedEventIndex"
            @focus="$event.target.select()">
          <span :hidden="i == selectedEventIndex">{{event.startTime.format('YY-MM-DD HH:mm')}}</span>
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
  import moment from 'moment-msdate';
  import uuidv4 from 'uuid/v4';

  const CLIENT_ID = '781056449218-gff1pg8oqs62ovk0et70mtgchjbp04l8.apps.googleusercontent.com';
  const API_KEY = 'AIzaSyBHbnon2LBBMsE6te0GgUNqFBrNElWApnY';
  const DISCOVERY_DOCS = ["https://sheets.googleapis.com/$discovery/rest?version=v4"];
  const SCOPES = "https://www.googleapis.com/auth/spreadsheets";
  const spreadsheetId = '1lDFO2rjOaff566CRaiqbT9PIoVd3XMRSe45i3Z8SOy0';

  function sheetsToJsEpoch(sheetSerialNo) {
    return Math.round((sheetSerialNo - (25567 + 2))*86400)*1000
  }
  function jsToSheetsEpoch(jsSerialNo) {
    return jsSerialNo / 86400 / 1000 + 25567 + 2
  }

  export default {
    name: 'app',
    data () {
      return {
        events: [],
        eventType: '',
        selectedEventIndex: null,
        isSignedIn: false
      }
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
        this.events.push(newEvent);
        this.selectEvent(null);
        this.sortEvents();
        this.createEvent(newEvent);
      },
      changeEvent(event, fieldName, value) {
        // change event with id event.id
        console.log('changeEvent(event, ',fieldName, ', ', value, ')');
        event[fieldName] = value;
        this.saveEvent(event);
      },
      createEvent(event) {
        console.log('createEvent(',JSON.stringify(event),')');
        console.log('jsepo', jsToSheetsEpoch(event.startTime.seconds()));
        var values = [
          [event.id, event.type, event.startTime.toOADate(), event.comment],
        ];
        var body = {
          values: values
        };
        gapi.client.sheets.spreadsheets.values.append({
          spreadsheetId,
          range: 'Class Data!A2:D',
          valueInputOption: 'RAW',
          resource: body
        }).then((response) => {
          var result = response.result;
          console.log(`${result.updates.updatedCells} cells appended.`)
        });
      },
      sortEvents() {
        this.events.sort((a, b)=>b.startTime.isBefore(a.startTime))
      },
      fetchEvents() {
        let self = this;
        gapi.client.sheets.spreadsheets.values.get({
          spreadsheetId,
          range: 'Class Data!A2:D',
          valueRenderOption: 'UNFORMATTED_VALUE',
          dateTimeRenderOption: 'SERIAL_NUMBER'
        }).then(function(response) {
          var range = response.result;
          if (range.values && range.values.length > 0) {
            let events = [];
            for (let i = 0; i < range.values.length; i++) {
              var row = range.values[i];
              if (row[0]) {
                console.log('fetched date', moment.fromOADate(row[2]));
                events.push({rowI: i + 2, id: row[0], type: row[1], startTime: moment.fromOADate(row[2], moment().utcOffset()), comment: row[3]})
              }
            }
            self.events = events;
            self.sortEvents();
          }
        }, function(response) {
          console.log('Error:', response.result.error.message);
        });
      },
      saveEvent(event) {
        // save event with id event.id
        //TODO read from sheet and check that the same id
        console.log('saveEvent(', event,')');
        var body = {
          values: [
            [event.id, event.type, event.startTime.toOADate(), event.comment],
          ]
        };
        gapi.client.sheets.spreadsheets.values.update({
          spreadsheetId,
          range: 'Class Data!A' + event.rowI + ':D',
          valueInputOption: 'RAW',
          resource: body
        }).then((response) => {
          var result = response.result;
          console.log('saved event', result);
        });
      },
      deleteEvent(event) {
        //TODO delete event with id event.id
      },
      moment,
      auth (immediate) {
        console.log('Start oauth...');
        gapi.auth.authorize(
          {
            'client_id': CLIENT_ID,
            'scope': SCOPES,
            'immediate': (typeof immediate === 'undefined')
          },
          this.handleAuthResult
        );
      },
      loadSheetAPI () {
        console.log('Loading sheet api...');
        gapi.client.load('https://sheets.googleapis.com/$discovery/rest?version=v4')
          .then(this.getAllSheets, (resp) => {
            console.log('Unable to load sheet api')
          });
      },
      initClient() {
        let self = this;
        gapi.client.init({
          apiKey: API_KEY,
          clientId: CLIENT_ID,
          discoveryDocs: DISCOVERY_DOCS,
          scope: SCOPES
        }).then(function () {
          console.log('gapi initialized');
          // Listen for sign-in state changes.
          gapi.auth2.getAuthInstance().isSignedIn.listen(self.updateSigninStatus);

          // Handle the initial sign-in state.
          self.updateSigninStatus(gapi.auth2.getAuthInstance().isSignedIn.get());
          // authorizeButton.onclick = handleAuthClick;
          // signoutButton.onclick = handleSignoutClick;
        });
      },

      updateSigninStatus(isSignedIn) {
        this.isSignedIn = isSignedIn;
        console.log('isSignedIn:', isSignedIn);
        if (isSignedIn) {
        //   authorizeButton.style.display = 'none';
        //   signoutButton.style.display = 'block';

          this.fetchEvents();
        } else {
        //   authorizeButton.style.display = 'block';
        //   signoutButton.style.display = 'none';
        }
      },
      handleAuthClick(event) {
        console.log('gapi.auth2.getAuthInstance()', gapi.auth2.getAuthInstance());
        gapi.auth2.getAuthInstance().signIn();
      },
      handleSignoutClick(event) {
        gapi.auth2.getAuthInstance().signOut();
      }
    },
    created() {
      console.log('created');
      // Expose global function
      window.onSheetApiLoaded = this.auth;
      gapi.load('client:auth2', this.initClient);
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
