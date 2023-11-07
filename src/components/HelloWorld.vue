<template>
  <!-- <h1>{{ msg }}</h1> -->
  <div class="container">
    <div style="text-align: center">
      <h1>Smart on FHIR - Patient Info</h1>
      <p><strong>Username:</strong> fhircamila</p>
      <p><strong>Password:</strong> epicepic1</p>
      <a
        class="btn btn-info"
        v-if="code == undefined"
        style="text-decoration: none"
        target="_blank"
        :href="authorizeLink"
      >
        Sign in
      </a>
      <hr />
    </div>
    <!-- <div v-if="accesstoken">
   
      <p><strong>Patient Id:</strong> {{ patient }}</p>
      <strong>Name: </strong>{{ patientdata.name[0].text }}<br />
      <strong>Birth Date: </strong>{{ patientdata.birthDate }} <br />
      <strong>Gender: </strong>{{ patientdata.gender }} <br />
      <strong>Vital Status: </strong
      >{{ patientdata.deceasedBoolean ? "Dead" : "Alive" }} <br />
      <strong>Marital Status: </strong>{{ patientdata.maritalStatus.text }}
      <br />
      <strong>Telecom: </strong> <br />
      <div v-for="telecom in patientdata.telecom" :key="telecom.value">
        <div class="ml-2">
          <strong>{{ telecom.system }}</strong> -
          {{ telecom.use }}
          {{ telecom.value }}
        </div>
      </div>
    <hr />
    <strong>Access code:</strong>
    <p class="ml-2" style="word-break: break-all">{{ accesstoken }}</p>
    <strong>Patient Resource:</strong>
    <pre>{{ patientdata }}</pre>
</div> -->
<div v-if="accesstoken">
    <p><strong>Age:</strong> {{ patientdata.birthDate }} <br /></p>
    <strong>Sex: </strong>{{ patientdata.gender === 'male' ? 'Male' : 'Female' }} <br />
    <strong>Vitals:</strong><br />
    <div class="ml-2">
        <strong>Blood Pressure:</strong> {{ patientdata.bloodPressure }} <br />
        <strong>Respiratory Rate:</strong> {{ patientdata.respiratoryRate }} <br />
        <strong>Heart Rate:</strong> {{ patientdata.heartRate }} <br />
        <strong>Oxygen Saturation:</strong> {{ patientdata.oxygenSaturation }} <br />
    </div>

    <strong>Labs:</strong><br />
    <div class="ml-2">
        <strong>Blood Urea Nitrogen (BUN):</strong> {{ patientdata.bloodUreaNitrogen }} <br />
        <strong>Creatinine:</strong> {{ patientdata.creatinine }} <br />
        <strong>Hemoglobin:</strong> {{ patientdata.hemoglobin }} <br />
    </div>

    <strong>Medications:</strong> {{ patientdata.medications }} <br />
    <strong>Diagnosis/Problem List:</strong> {{ patientdata.diagnosis }} <br />

    <strong>Interpretation of Data:</strong><br />
    <div class="ml-2">
        <strong>Example:</strong><br />
        <strong>Criteria:</strong> Age &gt; 65, BP &lt; 90, RR &gt; 30, BUN &gt; 19 and confusion <br />
        <strong>Interpretation:</strong> 
        <span v-if="patientdata.age &gt; 65 &amp;&amp; patientdata.bloodPressure &lt; 90 &amp;&amp; patientdata.respiratoryRate &gt; 30 &amp;&amp; patientdata.bloodUreaNitrogen &gt; 19 &amp;&amp; patientdata.confusion">3 or more out of 5 – Interpret as severe.</span>
        <span v-else>1-2 out of 5 – interpret as Mild.</span>
    </div>

    <hr />
    <strong>Access code:</strong>
    <p class="ml-2" style="word-break: break-all">{{ accesstoken }}</p>
    <strong>Patient Resource:</strong>
    <pre>{{ patientdata }}</pre>
</div>

  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      code: "",
      accesstoken: "",
      patient: "",
      patientdata: {},
      clientId: "a28e02c7-c28c-4443-ae2d-a577a1addacd", // Replace with your client id
      redirect: import.meta.env.PROD
        ? "https://lucid-wozniak-940eae.netlify.app"
        : "http://localhost:3000",
    };
  },
  computed: {
    authorizeLink() {
      return `https://fhir.epic.com/interconnect-fhir-oauth/oauth2/authorize?response_type=code&redirect_uri=${this.redirect}&client_id=${this.cl
        ientId}&state=1234&scope=patient.read, patient.search`;
    },
  },
  async mounted() {
    console.log(this.$route.query.code);
    this.code = this.$route.query.code;
    if (this.code != undefined) {
      const params = new URLSearchParams();
      params.append("grant_type", "authorization_code");
      params.append("redirect_uri", this.redirect);
      params.append("code", this.code);
      params.append("client_id", this.clientId);
      params.append("state", "1234");
      const config = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      };
      await axios
        .post(
          "https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token",
          params,
          config
        )
        .then((response) => {
          console.log(response.data);
          this.accesstoken = response.data.access_token;
          this.patient = response.data.patient;
        })
        .catch(function (error) {
          console.log(error);
        });
    }

    if (this.accesstoken != "") {
      await axios
        .get(
          `https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4/Patient/${this.patient}`,
          { headers: { Authorization: `Bearer ${this.accesstoken}` } }
        )
        .then((response) => {
          this.patientdata = response.data;
          console.log(response);
        })
        .catch((error) => {
          console.log(error);
        });
    }
  },
};
// const state = reactive({ count: 0 })
</script>

<style scoped>
/* a {
  color: #42b983;
} */
</style>
