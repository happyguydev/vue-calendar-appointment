<template>
  <div style="padding: 10px">
    <v-row class="fill-height">
      <v-col cols="9">
        <vue-cal today-button
                 hide-view-selector
                 sticky-split-labels
                 active-view="day"
                 ref="vuecal"
                 :time-from="0"
                 :time-to="24 * 60"
                 :time-step="15"
                 :time-cell-height="30"
                 :events="events"
                 :split-days="splitUsers"
                 :selected-date="selectedDate"
                 :disable-views="['years', 'year', 'month', 'week']"
                 :drag-to-create-event="false"
                 :cell-click-hold="false"
                 :min-split-width="150"
                 @cell-dblclick="onEventCreate">
        </vue-cal>
      </v-col>
      <v-col cols="3">
        <vue-cal class="vuecal--blue-theme vuecal--rounded-theme"
                 style="max-width: 270px;height: 290px"
                 active-view="month"
                 xsmall
                 hide-view-selector
                 :time="false"
                 :disable-views="['years', 'year', 'week', 'day']"
                 @cell-focus="selectedDate = $event">
        </vue-cal>
        <p class="mt-10">Resource</p>
        <div v-for="(value, name, index) in splitUsers" v-bind:key="index">
          <v-checkbox
              v-model="selectedResource"
              :label="`${value.label}`"
              :value="value.id"
          ></v-checkbox>
        </div>
      </v-col>
    </v-row>
    <v-dialog v-model="showEventCreationDialog" :persistent="true" max-width="800">
      <v-card>
        <v-card-title>
          New Appointment
        </v-card-title>
        <v-card-text>
          <v-text-field
              v-model="patientName"
              label="Patient Name"
              placeholder="Patient Name"
              :rules="nameRules"
              required
          />
          <v-row>
            <v-col
                class="d-flex align-center"
                cols="12"
                md="2"
            >
              <span>Start Time: </span>
            </v-col>
            <v-col
                cols="12"
                md="5"
            >
              <v-menu
                  ref="startDateMenu"
                  v-model="startDateMenu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  offset-y
                  max-width="290px"
                  min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                      v-model="startDateFormatted"
                      label="Date"
                      persistent-hint
                      prepend-icon="mdi-calendar"
                      v-bind="attrs"
                      @blur="startDate = parseDate(startDateFormatted)"
                      v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                    v-model="startDate"
                    no-title
                    @input="startDateMenu = false"
                ></v-date-picker>
              </v-menu>
            </v-col>

            <v-col
                cols="12"
                md="5"
            >
              <v-menu
                  ref="startTimeMenu"
                  v-model="startTimeModel"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  :return-value.sync="startTime"
                  transition="scale-transition"
                  offset-y
                  max-width="290px"
                  min-width="290px"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                      v-model="startTime"
                      label="Time"
                      prepend-icon="mdi-clock-time-four-outline"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                  ></v-text-field>
                </template>
                <v-time-picker
                    v-if="startTimeModel"
                    v-model="startTime"
                    full-width
                    @click:minute="$refs.startTimeMenu.save(startTime)"
                ></v-time-picker>
              </v-menu>
            </v-col>
          </v-row>
          <v-row class="mt-0">
            <v-col
                class="d-flex align-center"
                cols="12"
                md="2"
            >
              <span>End Time: </span>
            </v-col>
            <v-col
                cols="12"
                md="5"
            >
              <v-menu
                  ref="endDateMenu"
                  v-model="endDateMenu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  offset-y
                  max-width="290px"
                  min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                      v-model="endDateFormatted"
                      label="Date"
                      persistent-hint
                      prepend-icon="mdi-calendar"
                      v-bind="attrs"
                      @blur="endDate = parseDate(endDateFormatted)"
                      v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                    v-model="endDate"
                    no-title
                    @input="endDateMenu = false"
                ></v-date-picker>
              </v-menu>
            </v-col>

            <v-col
                cols="12"
                md="5"
            >
              <v-menu
                  ref="endTimeMenu"
                  v-model="endTimeModel"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  :return-value.sync="endTime"
                  transition="scale-transition"
                  offset-y
                  max-width="290px"
                  min-width="290px"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                      v-model="endTime"
                      label="Time"
                      prepend-icon="mdi-clock-time-four-outline"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                  ></v-text-field>
                </template>
                <v-time-picker
                    v-if="endTimeModel"
                    v-model="endTime"
                    full-width
                    @click:minute="$refs.endTimeMenu.save(endTime)"
                ></v-time-picker>
              </v-menu>
            </v-col>
          </v-row>
          <v-textarea
              v-model="notes"
              label="Notes"
              counter
              maxlength="200"
          ></v-textarea>
          <v-layout justify-end>
            <v-btn class="mr-4"
                   color="primary"
                   @click="closeCreationDialog()"
            >Save</v-btn>
            <v-btn @click="cancelEventCreation()">Cancel</v-btn>
          </v-layout>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import VueCal from 'vue-cal'
import 'vue-cal/dist/vuecal.css'

export default {
  name: 'CalendarAppointment',
  props: {
    msg: String
  },
  components: { VueCal },
  data:vm => ({
    events: [],
    splitUsers: [
      { id: 1, label: 'Dr 1' },
      { id: 2, label: 'Dr 2' },
      { id: 3, label: 'Dr 3' },
      { id: 4, label: 'Dr 4' },
      { id: 5, label: 'Dr 5' }
    ],
    selectedResource: [],
    showEventCreationDialog: false,
    selectedDate: null,
    selectedEvent: null,
    patientName: null,
    nameRules: [
      v => !!v || 'Patient Name is required',
    ],
    notes: null,

    startDateMenu: false,
    startDate: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
    startDateFormatted: vm.formatDate((new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10)),

    endDateMenu: false,
    endDate: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
    endDateFormatted: vm.formatDate((new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10)),

    startTimeModel: false,
    startTime: '09:00',

    endTimeModel: false,
    endTime: '09:00',
  }),
  watch: {
    startDate () {
      this.startDateFormatted = this.formatDate(this.startDate)
    },
    endDate () {
      this.endDateFormatted = this.formatDate(this.endDate)
    },
  },
  methods: {
    formatDate (date) {
      if (!date) return null

      const [year, month, day] = date.split('-')
      return `${month}/${day}/${year}`
    },
    parseDate (date) {
      if (!date) return null

      const [month, day, year] = date.split('/')
      return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
    },
    onEventCreate (event) {
      this.selectedEvent = event
      this.showEventCreationDialog = true

      return event
    },
    cancelEventCreation () {
      this.showEventCreationDialog = false
      this.selectedEvent = {}
    },
    closeCreationDialog () {
      if (
          this.startDate > this.endDate
          || this.startTime > this.endTime
          || !this.patientName
      ) {
        alert('Information incorrect!')
      } else {
        this.showEventCreationDialog = false
        this.events.push({
          start: this.startDate + ' ' + this.startTime,
          end: this.endDate + ' ' + this.endTime,
          title: this.patientName,
          content: this.notes,
          class: 'test-class',
          split: this.selectedEvent.split
        })
        this.selectedEvent = {}
      }
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .vuecal__no-event {
    display: none;
  }
  .vuecal__cell-split {
    border-left: 1px solid hsla(0,0%,76.9%,.25);
    background-color: rgba(255, 250, 196, 0.5);
  }
  .vuecal__event {
    background-color: rgb(255, 255, 255);
    border: 1px solid;
    cursor: pointer;
    user-select: none;
  }
</style>
