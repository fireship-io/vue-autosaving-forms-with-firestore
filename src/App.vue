<template>
  <div id="app">
    <div class="notification is-success" v-if="state === 'synced'">Form is synced with Firestore</div>
    <div
      class="notification is-link"
      v-else-if="state === 'modified'"
    >From data changed, will sync with Firebase</div>
    <div
      class="notification is-warning"
      v-else-if="state === 'revoked'"
    >From data and Firebase revoked to original data</div>
    <div class="notification is-danger" v-else-if="state === 'error'">Failed to save to Firestore</div>
    <div class="notification is-info" v-else>Waiting for changes...</div>

    <hr />
    <div class="columns">
      <div class="column">
        <h3>Original Data</h3>
        <br />
        {{ originalData }}
      </div>
    </div>
    <div class="columns">
      <div class="column">
        <h3>Vue Form Data</h3>
        <br />
        {{ formData }}
      </div>
      <div class="column">
        <h3>Firestore Data</h3>
        <br />
        {{ firebaseData }}
      </div>
    </div>

    <hr />

    <form @submit.prevent="saveForm">
      <div class="field is-horizontal">
        <div class="field-label is-normal">
          <label class="label">Contact</label>
        </div>
        <div class="field-body">
          <div class="field">
            <p class="control is-expanded has-icons-left">
              <input
                class="input"
                type="text"
                name="name"
                v-model="formData.name"
                @input="fieldUpdate"
              />
              <span class="icon is-small is-left">
                <i class="fas fa-user"></i>
              </span>
            </p>
          </div>
          <div class="field">
            <p class="control is-expanded has-icons-left has-icons-right">
              <input
                class="input"
                type="email"
                name="email"
                v-model="formData.email"
                @input="fieldUpdate"
              />
              <span class="icon is-small is-left">
                <i class="fas fa-envelope"></i>
              </span>
            </p>
          </div>
        </div>
      </div>
      <div class="field is-horizontal">
        <div class="field-label is-normal">
          <label class="label">Phone</label>
        </div>
        <div class="field-body">
          <div class="field is-expanded">
            <div class="field has-addons">
              <p class="control">
                <a class="button is-static">+44</a>
              </p>
              <p class="control is-expanded">
                <input
                  class="input"
                  type="tel"
                  name="phonenumber"
                  v-model="formData.phonenumber"
                  @input="fieldUpdate"
                />
              </p>
            </div>
            <p class="help">Do not enter the first zero</p>
          </div>
        </div>
      </div>
      <hr />
      Vue Form State: {{ state == '' ? "waiting for changes" : state }}
      <hr />
    </form>
    <button class="button is-warning is-rounded" @click="revokeData">
      <span class="icon">
        <i class="fas fa-undo"></i>
      </span>
      <span>Revoke to original data</span>
    </button>
  </div>
</template>

<script lang="ts">
import { db } from "./firebase";
import { debounce } from "debounce";

export default {
  data() {
    return {
      state: "",
      originalData: {},
      formData: {},
      firebaseData: {}
    };
  },

  firestore() {
    return {
      firebaseData: db.doc("documents/contact")
    };
  },

  methods: {
    updateFirebase: debounce(async function() {
      try {
        this.state = "synced";
        await db.doc("documents/contact").set(this.formData);
      } catch (error) {
        this.state = "error";
      }
    }, 1500),

    revokeData() {
      const _that = this as any;
      this.formData = (_that as any).originalData;
      this.updateFirebase();
      this.state = "revoked";
    },

    getDataOnce: async function() {
      const data = await db.doc("documents/contact").get();
      return data.data();
    },

    fieldUpdate() {
      this.state = "modified";
      this.updateFirebase();
    },
  },

  created: async function() {
    (this as any).originalData = await (this as any).getDataOnce();
    (this as any).formData = await (this as any).getDataOnce();
  }
};
</script>

<style>
h1,
h2,
h3 {
  font-weight: 600;
}

#app {
  font-family: "sofia-pro", sans-serif;
  text-align: center;
  padding: 10vh 15vw;
}

p.help {
  text-align: left;
}
</style>
