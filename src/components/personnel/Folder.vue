<template>
  <div>
  <v-data-table
    v-model="selected"
    :headers="headers"
    :items="folder"
    :single-select="singleSelect"
    item-key="cntrl_no"
    show-select
    class="elevation-2"
    :loading="loading"
  >
    <template v-slot:top>
      <v-toolbar flat color="white">
        <v-switch v-model="singleSelect" label="Single select" class="pa-3 mt-3"></v-switch>
        <v-spacer></v-spacer>
        <v-btn color="primary" dark class="mb-2" small>Switch to Employees</v-btn>
      </v-toolbar>
    </template>
    <template v-slot:item.day="{ item }">
      {{ getDay(item.day) }}
    </template>
    <template v-slot:item.descript="{ item }">
    <v-edit-dialog
      @save="saveDayType"
      @cancel="cancelDayType"
      @open="openDayType"
      @close="closeDayType"
    > {{ item.descript }}
      <template v-slot:input>
        <v-select
          v-model="item.day_type"
          :items="dayType"
          item-text="descript"
          item-value="cntrl_no"
          label="Choose Day Type"
          @change="getSelectedDayType(item)"
        ></v-select>
      </template>
    </v-edit-dialog>
    </template>
    <template v-slot:item.action="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        edit
      </v-icon>
    </template>
  </v-data-table>
  <v-snackbar v-model="snack" :timeout="3000" :color="snackColor">
    {{ snackText }}
    <v-btn text @click="snack = false">Close</v-btn>
  </v-snackbar>
  </div>
</template>
<script>
import axios from 'axios'
import { Form } from 'vform'
var moment = require('moment')

export default {
  name: 'Folder',
  data () {
    return {
      primekey: '',
      snackColor: '',
      snackText: '',
      folder: [],
      dayType: [],
      selected: [],
      loading: false,
      snack: false,
      singleSelect: false,
      form: new Form({
        primekey: localStorage.getItem('primekey'),
        cntrl_no: '',
        day_type: ''
      }),
      headers: [
        {
          text: 'Folder #',
          align: 'left',
          sortable: true,
          value: 'cntrl_no'
        },
        { text: 'DTR Date', value: 'dtr_date' },
        { text: 'Day', value: 'day' },
        { text: 'Day Type', value: 'descript' },
        { text: 'Created At', value: 'creat_dt' },
        { text: 'Actions', value: 'action', sortable: false, align: 'center' }
      ]
    }
  },
  methods: {
    retrieveFolder () {
      // accept parameter for retrieve
      this.loading = true
      this.$store.dispatch('retrieveFolder', {
        primekey: localStorage.getItem('primekey'),
        cntrl_no: this.$route.params.cntrl_no
      })
        .then(response => {
          this.folder = this.$store.getters.retrieveFolder
          this.loading = false
        })
    },
    retrieveDayType () {
      // display all day type
      this.$store.dispatch('retrieveDayType', {
        primekey: localStorage.getItem('primekey')
      })
        .then(response => {
          this.dayType = this.$store.getters.retrieveDayType
        })
    },
    async saveDayType () {
      // update day type of folder
      try {
        axios.defaults.headers.common['Authorization'] = 'Bearer ' + localStorage.getItem('access_token')
        if (this.$store.getters.loggedIn) {
          await new Promise((resolve, reject) => {
            this.form.patch('u/personnel/directory/folder/daytype', {
            })
              .then(response => {
                resolve(response)
                this.$emit('retrieveFolder')
              })
              .catch(error => {
                reject(error)
              })
          })
        }
      } catch (error) {
      }
      this.snack = true
      this.snackColor = 'success'
      this.snackText = 'Data saved'
    },
    getDay (value) {
      return moment(value).format('dddd')
    },
    getSelectedDayType (value) {
      this.form.cntrl_no = value.cntrl_no
      this.form.day_type = value.day_type
    },
    cancelDayType () {
      this.snack = true
      this.snackColor = 'error'
      this.snackText = 'Canceled'
    },
    openDayType () {
      this.snack = true
      this.snackColor = 'info'
      this.snackText = 'Dialog opened'
    },
    closeDayType () {
      this.saveDayType()
    }
  },
  created () {
    this.retrieveFolder()
    this.retrieveDayType()
    this.$on('retrieveFolder', () => {
      this.retrieveFolder()
    })
  }
}
</script>
