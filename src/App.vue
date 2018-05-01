<template>
  <div id="app">
    <div class="events" @click="selectedEventId = null">
      <div class="">
        <button type="button" @click="handleAuthClick" :disabled="isSignedIn">login</button>
        <button type="button" @click="handleSignoutClick" :disabled="!isSignedIn">logout</button>
      </div>
      <div class="event" v-for="(event, i) in eventsLatestFirst"
        @click="selectEvent(event.id); $event.stopPropagation()"
        :class="{selected: event.id === selectedEventId}">
        <div class="row1">
          <input type="text" :value="event.type" @change="changeEvent(event, 'type', $event.target.value)"
            :hidden="event.id !== selectedEventId"
            @focus="$event.target.select()">
          <span :hidden="event.id === selectedEventId">{{event.type}}</span>
          <span class="filler"></span>
          <input type="text" :value="formatDate(event.startTime, 'YY-MM-DD HH:mm')"
            @change="changeEvent(event, 'startTime', parseDateStd($event.target.value))"
            :hidden="event.id !== selectedEventId"
            @focus="$event.target.select()">
          <span :hidden="event.id === selectedEventId">{{formatDate(event.startTime, 'YY-MM-DD HH:mm')}}</span>
        </div>
        <div class="row2">
          <input type="text"
            :value="event.comment"
            @change="changeEvent(event, 'comment', $event.target.value)"
            :hidden="event.id !== selectedEventId"
            @focus="$event.target.select()">
          <span :hidden="event.id === selectedEventId">{{event.comment}}</span>
        </div>
      </div>
      <div class="event dummy-event">
      </div>
    </div>
    <div class="event-type-filter">
      <input type="text" v-model="eventType">
      <button type="button" @click="addEvent(eventType)" :disabled="eventType.trim() === ''">Start</button>
    </div>
    <div class="event-types">
      <div class="event-type" v-for="eventType in eventTypes" @click="addEvent(eventType)">
        {{eventType}}
      </div>
    </div>
  </div>
</template>

<script>
  import formatDate from 'date-fns/format';
  import parseDate from 'date-fns/parse';
  import compareDateAsc from 'date-fns/compare_asc';
  import uuidv4 from 'uuid/v4';

  const CLIENT_ID = '781056449218-gff1pg8oqs62ovk0et70mtgchjbp04l8.apps.googleusercontent.com';
  const DISCOVERY_DOCS = ["https://sheets.googleapis.com/$discovery/rest?version=v4"];
  const SCOPES = "https://www.googleapis.com/auth/spreadsheets";
  const spreadsheetId = '1lDFO2rjOaff566CRaiqbT9PIoVd3XMRSe45i3Z8SOy0';

  function sheetsToJsEpoch(sheetSerialNo) {
    return (sheetSerialNo - (25567 + 2))*86400*1000
  }
  function jsToSheetsEpoch(jsSerialNo) {
    return jsSerialNo / 86400 / 1000 + 25567 + 2
  }

  function compareEventsByStartDate(a, b) {
    return compareDateAsc(a.startTime, b.startTime)
  }

  export default {
    name: 'app',
    data () {
      return {
        events: [],
        eventType: '',
        selectedEventId: null,
        isSignedIn: false
      }
    },
    computed: {
      eventsLatestFirst() {
        return [...this.events].sort((a, b)=>compareEventsByStartDate(b, a))
      },
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
      formatDate,
      parseDateStd(s) {
        return parseDate('20' + s.replace(' ', 'T'))
      },
      selectEvent(id) {
        console.log('selectEvent(',id,')');
        if (this.selectedEventId === id) {
          //this.selectedEventIndex = null;
        } else {
          this.selectedEventId = id;
        }
      },
      addEvent(eventType) {
        let newEvent = {id: uuidv4(), type: eventType, startTime: new Date(), comment: ''};
        this.events.push(newEvent);
        this.selectEvent(null);
        this.events.sort(compareEventsByStartDate);
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
        var values = [
          [event.id, event.type, jsToSheetsEpoch(event.startTime), event.comment],
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
      fetchEvents() {
        console.log('fetchEvents');
        let self = this;
        gapi.client.sheets.spreadsheets.values.get({
          spreadsheetId,
          range: 'Class Data!A2:D',
          valueRenderOption: 'UNFORMATTED_VALUE',
          dateTimeRenderOption: 'SERIAL_NUMBER'
        }).then(function(response) {
          console.log('fetchEvents response');
          var range = response.result;
          if (range.values && range.values.length > 0) {
            let events = [];
            for (let i = 0; i < range.values.length; i++) {
              var row = range.values[i];
              if (row[0]) {
                events.push({rowI: i + 2, id: row[0], type: row[1], startTime: new Date(sheetsToJsEpoch(row[2])), comment: row[3]})
              }
            }
            self.events = events.sort(compareEventsByStartDate);
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
            [event.id, event.type, jsToSheetsEpoch(event.startTime), event.comment],
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
