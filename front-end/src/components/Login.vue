<template>
  <div class="Login">
    <div class="initial-screen" v-if="this.view == 0">
      <h1>Login to Make and Manage Appointments</h1>
      <button @click="changeView(1)">Login</button>
      <button @click="changeView(2)">Create Account</button>
    </div>

    <div class="login-form" v-if="this.view == 1">
      <h1>Login</h1>
      <input v-model="loginUserName" placeholder="Username" />
      <input v-model="loginPassword" placeholder="Password" type="password" />
      <div class="alert alert-danger banner" role="alert" v-if="errorLogin">
        Incorrect Username or Password
      </div>
      <button @click="login()">Login</button>
    </div>

    <div class="create-account-form" v-if="this.view == 2">
      <h1>Hello, new user!</h1>
      <input v-model="firstName" placeholder="First Name" />
      <input v-model="lastName" placeholder="Last Name" />
      <input v-model="email" placeholder="Email" />
      <input v-model="userName" placeholder="Username" />
      <input v-model="password" placeholder="Password" type="password" />

      <div class="alert alert-danger banner" role="alert" v-if="error">
        Username Already Exists
      </div>

      <button @click="createNewAccount()">Create Account</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'HomePage',
  data() {
    return {
      view: "0",
      userName: "",
      password: "",
      firstName: "",
      lastName: "",
      email: "",
      error: "",
      errorLogin: "",
      loginUserName: "",
      loginPassword: "",
    }
  },
  methods: {
    changeView(newView) {
      this.view = newView;
    },
    async createNewAccount() {
      this.error = '';
      this.errorLogin = '';
      if (this.firstName == ""|| this.lastName == ""|| this.email == ""|| this.username == "" || this.password == "")
        return;
      try {
        let response = await axios.post('/api/users', {
          firstName: this.firstName,
          lastName: this.lastName,
          email: this.email,
          username: this.userName,
          password: this.password,
        });
        this.$root.$data.user = response.data.user;
      } catch (error) {
        this.error = error.response.data.message;
        //console.log(this.error);
        this.$root.$data.user = null;
      }
    },
    async login() {
      this.error = '';
      this.errorLogin = '';
      if (this.loginUserName  == "" || this.loginPassword == "")
        return;
      try {
        let response = await axios.post('/api/users/login', {
          username: this.loginUserName,
          password: this.loginPassword,
          });
          this.$root.$data.user = response.data.user;
          if(this.$root.$data.user.role == null) {
            this.view = 3;
            location.reload(); //Reloads the page
          }
          else {
            location.reload(); //Reloads the page
            this.$router.push("Admin");
          }
      } catch (error) {
        this.errorLogin = "Error: " + error.response.data.message;
        this.$root.$data.user = null;
      }
   },
  },
}
</script>

<style scoped>

* {
  margin: 0;
  padding: 0;
}

.initial-screen {
  margin-top: 200px;
  border: black outset thin;
  width: 70%;
  margin-left: auto;
  margin-right: auto;
  display: block;
}

.initial-screen button {
  width: 70%;
  margin-top: 10px;
  margin-bottom: 10px;
}

.login-form {
  margin-top: 200px;
  border: outset black thin;
  width: 70%;
  margin-left: auto;
  margin-right: auto;
  display: block;
}

.login-form input {
  width: 70%;
  margin-top: 10px;
  margin-bottom: 10px;

}

.login-form button {
  width: 70%;
  margin-top: 10px;
  margin-bottom: 10px;
}

.create-account-form {
  margin-top: 200px;
  display: block;
  width: 70%;
  margin-left: auto;
  margin-right: auto;
  border: outset black thin;
  padding-top: 10px;
  padding-left: 5px;
  padding-right: 5px;
}

.create-account-form input {
  width: 70%;
  margin-top: 10px;
  margin-bottom: 5px;
}

.create-account-form button {
  width: 70%;
  margin-top: 10px;
  margin-bottom: 10px;
}

.banner {
  width: 70%;
  margin-left: auto;
  margin-right: auto;
  padding: 5px 5px 5px 5px;
}

/* Desktop Styles */
@media only screen and (min-width: 961px) {

.initial-screen {
  width: 25%;
  padding-top: 10px;
  padding-left: 10px;
  padding-right: 10px;
  padding-bottom: 10px;
  font-size: 25px;
}

.initial-screen button {
  background-color: #3771D8;
  color: #FFFFFF;
}

.login-form {
  width: 25%;
}

.login-form  button {
  font-size: 25px;
  margin-top: 20px;
  background-color: #3771D8;
  color: #FFFFFF;
}

.login-form input {
  font-size: 25px;
}

.login-form h1 {
  margin-bottom: 20px;
}

.create-account-form {
  width: 25%;
  font-size: 25px;
}

.create-account-form h1 {
  margin-bottom: 20px;
}

.create-account-form button {
  margin-top: 30px;
  margin-bottom: 20px;
  background-color: #3771D8;
  color: #FFFFFF;
}


}

</style>
