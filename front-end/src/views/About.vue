<template>
  <div class="about">
    <div v-if="this.$root.$data.user != null">
      <div id="page-title" v-if="this.appointmentList.length != 0">
        <h1>Appointment Manager</h1>
      </div>

      <div id="message" v-if="this.appointmentList.length == 0 && this.$root.$data.user.role == null">
        <h1><strong>No Scheduled Appointments...</strong></h1>
        <p>Click to <router-link to="/">here</router-link> to schedule an appointment</p>
      </div>

      <div id="message" v-if="this.appointmentList.length == 0 && this.$root.$data.user.role != null">
        <h1><strong>Now Signed in as Administor</strong></h1>
        <i class="fas fa-user-shield fa-7x"></i>
        <h5 id="user-header">You can go to the following pages:</h5>
        <button @click="goHome" class="adminButton">Home</button>
        <button @click="goAdmin" class="adminButton">Admin</button>
      </div>

      <div class="listOfAppointments" v-if="this.appointmentList.length != 0">
        <div v-for="appointment in appointmentList" :key="appointment._id" class="single-appointment">
          <h1 class="element">{{appointment.placeName}}</h1>
          <h3 class="element"> <i class="fas fa-map-marker-alt blue"></i> {{appointment.placeAddress}}, {{appointment.placeCity}} {{appointment.placeZipcode}}</h3>
          <h5 class="element"> <i class="far fa-calendar-alt"></i> {{appointment.appointmentDate}}</h5>
          <h5 class="element"> <i class="far fa-clock"></i> {{appointment.appointmentTime}}</h5>
          <h6 class="element"> <i class="far fa-user"></i> {{appointment.user.firstName}} {{appointment.user.lastName}}</h6>
          <h6>Confirmation Email Sent To: {{appointment.user.email}}</h6>
          <h6 class="apptNotdone" v-if="appointment.completed == false">Status: Awaiting Vaccination Day</h6>
          <h6 class="apptDone" v-if="appointment.completed == true">Status: Done <i class="fas fa-syringe"></i></h6>

          <div class="button-div">
            <button id="edit" @click="editAppointmentFunction(appointment._id)" v-if="editAppointment == false && appointment.completed == false">Edit</button>
            <button id="cancel" @click="cancelAppt(appointment._id)" v-if="editAppointment == false && appointment.completed == false">Cancel</button>
            <button id="done" @click="apptDone(appointment._id)">Mark Done</button>
          </div>

          <div v-if="show(appointment._id)" class="edit-apt-div">
            <label>Enter new changes</label>
            <div class="first-div">
              <input type="date" v-model="newDate" placeHolder="Appt. Date: mm-dd-yyyy"/>
              <select class="date-input" @change="changeAppointmentTimeAgain($event)">
                <option value="" selected disabled>Select New Time</option>
                <option value="8:00 am">8:00 am </option>
                <option value="9:00 am">9:00 am </option>
                <option value="10:00 am">10:00 am </option>
                <option value="11:00 am">11:00 am </option>
                <option value="12:00 pm">12:00 pm </option>
                <option value="1:00 pm">1:00 pm </option>
                <option value="2:00 pm">2:00 pm </option>
                <option value="3:00 pm">3:00 pm </option>
                <option value="4:00 pm">4:00 pm </option>
                <option value="5:00 pm">5:00 pm </option>
              </select>
            </div>

            <button @click="saveAppointmentEdits(appointment._id, appointment.placeName, appointment.placeAddress, appointment.placeCity, appointment.placeZipcode)">Save Changes</button>
            <button id="nevermind" @click="cancelEditAppt()">Delete Changes</button>
          </div>

        </div>
        <div id="helpful">
          <h1>Things To Know </h1>
          <ul>
            <li>Make sure to bring personal identfication documents(Driver's license, passport, work ID)</li>
            <li>Have your appointment confirmation available when checking-in</li>
            <li>Arrive 15 minutes ahead of your scheduled vaccination time</li>
          </ul>
        </div>

      </div>
    </div>
    <Login v-else />
  </div>
</template>


<script>
import axios from 'axios';
import Login from '@/components/Login.vue';
export default {
  name: 'About',
  components: {
    Login,
  },
  data() {
    return {
      appointmentList: [],
      editAppointment: false,
      editAppointmentID: "",
      newName: "",
      newAge: "",
      newTime: "",
      newDate: "",
    }
  },
  //created() {
    //this.getAppointmentList();
  //},
  async created() {
    this.getAppointmentList();
    try {
      let response = await axios.get('/api/users');
      this.$root.$data.user = response.data.user;
    } catch (error) {
      this.$root.$data.user = null;
    }
  },
  methods: {
    async getAppointmentList() {
      let userID = await axios.get('/api/users');
      userID = userID.data.user._id;
      //console.log(userID);
      let list = await axios.get('/api/appointment/' + userID);
      this.appointmentList = list.data;
    },
    async cancelAppt(id) {
      try {
        await axios.delete("/api/appointment/" + id);
        this.getAppointmentList();
        return true;
      } catch (error) {
        //console.log(error);
      }
    },
    editAppointmentFunction(id) {
      this.editAppointment = true;
      this.editAppointmentID = id;
    },
    cancelEditAppt() {
      this.editAppointment = false;
      this.editAppointmentID = "";
    },
    show(givenID) {
      if(givenID == this.editAppointmentID) {
        return true;
      }
      else {
        return false;
      }
    },
    changeAppointmentTimeAgain(event) {
      this.newTime = event.target.options[event.target.options.selectedIndex].text
    },
    async saveAppointmentEdits(id, place, address, city, zip) {
      try {
        await axios.put(`/api/appointment/${id}`, {
          appointmentTime: this.newTime,
          appointmentDate: this.newDate,
          placeName: place,
          placeAddress: address,
          placeZipcode: zip,
          placeCity: city
        });
        location.reload();

        this.newZipcode = "";
      } catch (error) {
        //console.log(error);
      }
    },
    async apptDone(apptID) {
      await axios.put('/api/appointment', {
        id: apptID,
        completed: true,
      });
      location.reload();
    },
    goHome() {
      this.$router.push("/");
    },
    goAdmin() {
      this.$router.push("/admin");
    }
  },
}
</script>



<style scoped>
  .about {
    min-height: calc(100vh - 80px - 100px);
    font-family: 'Quicksand', sans-serif;
    text-align: center;
  }

#page-title {
  margin-top: 30px;
}


.listOfAppointments {
  width: 85%;
  margin-left: auto;
  margin-right: auto;
}

.single-appointment {
  text-align: left;
  border: outset;
  padding: 10px 10px 10px 10px;
  margin-bottom: 20px;
}

.blue {
  color: #3771D8;
}

#edit {
  margin-right: 10px;
  height: 45px;
  width: 100px;
  background-color: #3771D8;
  color: #FFFFFF;
}

.button-div {
  margin-top: 30px;
}

#cancel {
  height: 45px;
  width: 100px;
  background-color: #CF2E17;
  color: #FFFFFF;
}

.element {
  margin-bottom: 20px;
}

#helpful {
  text-align: left;
  border: outset;
  list-style-position: inside;
  padding-left: 10px;
}

#message {
  margin-top: 40%;
}

.edit-apt-div {
  margin-top: 20px;
}

.first-div input {
  width: 60%;
  height: 30px;
  margin-bottom: 10px;
}

.first-div select {
  height: 30px;
  width: 60%;
  margin-bottom: 10px;
}

.second-div input {
  width: 60%;
  height: 30px;
  margin-bottom: 10px;
}

.edit-apt-div button {
  width: 60%;
  height: 45px;
  background-color: #2DAE46;
  color: #FFFFFF;
}

#nevermind {
  margin-top: 10px;
  background-color: #CF2E17;
  color: #FFFFFF;
}

#done {
  background-color: #2DAE46; /*green*/
  color: #FFFFFF;
  height: 45px;
  width: 100px;
  margin-left: 10px;
}

.apptNotdone {
  color: #CF2E17;
}

.apptDone {
  color: #2DAE46;
}

#user-header {
  margin-top: 20px;
  margin-bottom: 20px;
}

.adminButton {
  width: 100px;
  height: 50px;
  margin-left: 10px;
}

/* Desktop Styles */
@media only screen and (min-width: 961px) {

.listOfAppointments {
  width: 70%;
}

#message {
  margin-top: 20%;
}



} /*Closing bracket for media queries*/


</style>
