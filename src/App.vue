<template>
  <div id="app" class="my-app">
    <form class="my-form" v-if="!loggedIn && !showNewPassword" @submit.prevent="login">
      <h2>ログイン</h2>

      <div class="form-group">
        <label for="username">Username</label>
        <input class="form-control" type="text" id="username" required autocomplete="off" v-model="username">
        <small class="form-text text-muted">{{ username }}</small>
      </div>

      <div class="form-group">
        <label for="password">Password</label>
        <input class="form-control" type="password" id="password" required autocomplete="off" v-model="password">
        <small class="form-text text-muted">{{ password }}</small>
      </div>

      <button type="submit" class="btn btn-primary">Login</button>
    </form>

    <a href="#" class="btn btn-primary" role="button" v-if="loggedIn" @click="logout">Logout</a>

    <form class="my-form" v-if="showNewPassword" @submit.prevent="completeNewPassword">
      <h2>初期パスワード変更</h2>
      <div class="form-group">
        <label for="newPassword">Password</label>
        <input class="form-control" type="password" id="newPassword" required autocomplete="off" v-model="newPassword">
        <small class="form-text text-muted">{{ newPassword }}</small>
      </div>
      <button type="submit" class="btn btn-primary">New Password</button>
    </form>

    <form class="my-form" @submit.prevent="loadSession">
      <h2>セッションロード</h2>
      <button type="submit" class="btn btn-primary">Load session</button>
    </form>

    <form class="my-form" @submit.prevent="changePassword">
      <h2>パスワード変更</h2>
      <div class="form-group">
        <label for="previousPassword">Previous Password</label>
        <input class="form-control" type="password" id="previousPassword" required autocomplete="off" v-model="previousPassword">
        <small class="form-text text-muted">{{ previousPassword }}</small>
      </div>
      <div class="form-group">
        <label for="proposedPassword">Proposed Password</label>
        <input class="form-control" type="password" id="proposedPassword" required autocomplete="off" v-model="proposedPassword">
        <small class="form-text text-muted">{{ proposedPassword }}</small>
      </div>
      <button type="submit" class="btn btn-primary">Change Password</button>
    </form>
  </div>
</template>

<script>
import { CognitoUserPool, AuthenticationDetails, CognitoUser } from 'amazon-cognito-identity-js';

const USER_POOL_ID = '';
const CLENT_ID = '';

export default {
  name: 'app',

  data() {
    return {
      userPool: new CognitoUserPool({
        UserPoolId: USER_POOL_ID,
        ClientId: CLENT_ID
      }),
      cognitoUser: null,
      loggedIn: false,
      showNewPassword: false,
      username: '',
      password: '',
      newPassword: '',
      previousPassword: '',
      proposedPassword: ''
    }
  },

  mounted() {
    this.loadSession();
  },

  methods: {
    login(event) {
      var vm = this;
      this.cognitoUser = new CognitoUser({
        Username: this.username,
        Pool: this.userPool
      });
      var authenticationDetails = new AuthenticationDetails({
        Username: this.username,
        Password: this.password,
      })
      console.log(event);
      console.log(this.userPool);
      console.log(this.cognitoUser);
      console.log(authenticationDetails);
      this.cognitoUser.authenticateUser(authenticationDetails, {
        onSuccess: function(result) {
          console.log(result);
          console.log('access token + ' + result.getAccessToken().getJwtToken());
        },

        onFailure: function(err) {
          alert(err);
          console.log(JSON.stringify(err));
        },

        newPasswordRequired: function(userAttributes, requiredAttributes) {
          console.log('newPasswordRequired');
          vm.showNewPassword = true;
          console.log(vm.showNewPassword);
        }
      });
    },

    logout(event) {
      this.cognitoUser = this.userPool.getCurrentUser();
      if (this.cognitoUser == null) {
        return;
      }

      this.cognitoUser.signOut();
      this.loggedIn = false;
    },

    completeNewPassword(event) {
      var vm = this;
      console.log(event);
      console.log(this.cognitoUser);
      this.cognitoUser.completeNewPasswordChallenge(this.newPassword, null, {
        onSuccess: function(result) {
          console.log('onSuccess');
          console.log(result);
          console.log('access token + ' + result.getAccessToken().getJwtToken());
          vm.showNewPassword = false;
          vm.loggedIn = true;
        },

        onFailure: function(err) {
          alert(err);
          console.log(JSON.stringify(err));
        },
      });
    },

    loadSession() {
      var vm = this;

      this.cognitoUser = this.userPool.getCurrentUser();
      console.log(this.cognitoUser);

      if (this.cognitoUser === null) {
        this.loggedIn = false;
        return;
      }

      this.cognitoUser.getSession((err, session) => {
        if (err) {
          console.log(err);
          this.loggedIn = false;
          return;
        }

        if (session === null) {
          this.loggedIn = false;
        }

        this.loggedIn = true;
      });
    },

    changePassword(event) {
      var vm = this;
      console.log(event);
      this.cognitoUser = this.userPool.getCurrentUser();
      this.cognitoUser.getSession((err, session) => {
        if (err) {
          console.log(err);
          return;
        }

        console.log(this.cognitoUser);
        this.cognitoUser.changePassword(this.previousPassword, this.proposedPassword, function(err, result) {
          if (err) {
            console.log(err);
            console.log(JSON.stringify(err));
            return;
          }
          console.log(result);
        });
      });
    },
  }
}
</script>

<style lang="scss">
.my-app {
  padding-top: 20px;
}

.my-form {
  padding: 40px 0;
}
</style>
