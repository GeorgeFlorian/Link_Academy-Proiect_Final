<template>
  <section>
    <div class="top_container">
      <div class="mid_container">
        <div class="title">
          <h1>
            Setup - Change
            <span>Static IP</span> Settings
          </h1>
        </div>
        <div class="inner_container">
          <form
            method="POST"
            name="simple_form"
            @submit.prevent="`${wifi ? sendWifi() : sendEthernet()}`"
          >
            <div class="input_radio">
              <span>Please select the type of connection:</span>
              <br />
              <br />
              <input id="wifi" type="radio" name="connectionType" value="WiFi" />
              <label for="wifi" @click="wifi= true; ethernet= true; message=''">WiFi</label>
              <input id="eth" type="radio" name="connectionType" value="Ethernet" />
              <label for="eth" @click="wifi= false; ethernet= true; message=''">Ethernet</label>
            </div>
            <div id="wifiFields" v-if="wifi">
              <div class="input_row">
                <input
                  type="text"
                  class="input_text"
                  placeholder="Type here the Network Name (SSID)"
                  id="networkName"
                  name="networkName"
                  pattern=".{5,30}"
                  title="Enter between 5 and 30 characters"
                  required
                  v-model.lazy="ssid"
                />
                <label class="label_" for="networkName">Network Name (SSID)</label>
              </div>
              <div class="input_row">
                <input
                  type="password"
                  class="input_text"
                  placeholder="Type here the Password"
                  id="networkPassword"
                  name="networkPassword"
                  minlength="8"
                  pattern=".{8,63}"
                  title="Enter between 8 and 63 characters"
                  required
                  v-model.lazy="password"
                />
                <label class="label_" for="networkPassword">Password</label>
              </div>
            </div>
            <div id="ethernetFields" v-if="ethernet">
              <div class="input_row">
                <input
                  type="text"
                  class="input_text"
                  placeholder="Type here IP Address"
                  id="IP"
                  name="IP"
                  required
                  v-model.lazy="ipAddress"
                  @change="validateAdressOnChange"
                />
                <label class="label_" for="IP">IP Address</label>
              </div>
              <div class="input_row">
                <input
                  type="text"
                  class="input_text"
                  placeholder="Type here Gateway"
                  id="Gateway"
                  name="Gateway"
                  required
                  v-model.lazy="gateway"
                  @change="validateAdressOnChange"
                />
                <label class="label_" for="Gateway">Gateway</label>
              </div>
              <div class="input_row">
                <input
                  type="text"
                  class="input_text"
                  placeholder="Type here Subnet Mask"
                  id="SubnetMask"
                  name="SubnetMask"
                  required
                  v-model.lazy="subnet"
                  @change="validateAdressOnChange"
                />
                <label class="label_" for="SubnetMask">Subnet Mask</label>
              </div>
              <div class="input_row">
                <input
                  type="text"
                  class="input_text"
                  placeholder="Type here DNS"
                  id="DNS"
                  name="DNS"
                  required
                  v-model.lazy="dns"
                  @change="validateAdressOnChange"
                />
                <label class="label_" for="DNS">DNS</label>
              </div>
            </div>
            <input class="button" type="submit" value="Save Values" v-if="wifi || ethernet" />
          </form>
          <div class="message" v-if="message">{{ message }}</div>
          <div class="errors" v-if="errors.length > 0">
            <div v-for="error in errors" :key="error.id">{{error}}</div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { mapActions } from "vuex";
import axios from "axios";

const serverURL = location.origin;
const server = axios.create({ baseURL: serverURL, timeout: 5000 });

export default {
  name: "Static",
  data() {
    return {
      wifi: false,
      ethernet: false,
      ssid: "",
      password: "",
      ipAddress: "",
      gateway: "",
      subnet: "",
      dns: "",
      message: "",
      errors: [],
    };
  },
  methods: {
    ...mapActions(["addLog"]),
    netmaskToCidr(netmask) {
      return netmask.split(".").reduce((c, o) => c - Math.log2(256 - +o), 32);
    },
    getTime() {
      const today = new Date();
      let HH = today.getHours();
      HH = ("0" + HH).slice(-2);
      let MM = today.getMinutes();
      MM = ("0" + MM).slice(-2);
      let SS = today.getSeconds();
      SS = ("0" + SS).slice(-2);
      const time = `${HH}:${MM}:${SS}`;
      return time;
    },
    validateAdressOnChange(event) {
      var ipformat = /^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
      let value = event.target.value;
      let inputName = event.target.name;

      if (value == "") {
        this.$el.querySelector(`#${inputName}`).classList.remove("correct");
        this.$el.querySelector(`#${inputName}`).classList.remove("wrong");
        let index = this.errors.indexOf(
          `You have entered an invalid ${inputName} Address!`
        );
        if (index > -1) this.errors.splice(index, 1);
      } else if (value.match(ipformat) == null) {
        this.$el.querySelector(`#${inputName}`).classList.remove("correct");
        this.$el.querySelector(`#${inputName}`).classList.add("wrong");
        // this.$el.querySelector(`#${inputName}`).focus();
        let error = `You have entered an invalid ${inputName} Address!`;
        if (!this.errors.includes(error)) this.errors.push(error);
      } else if (value != null) {
        this.$el.querySelector(`#${inputName}`).classList.remove("wrong");
        this.$el.querySelector(`#${inputName}`).classList.add("correct");
        let index = this.errors.indexOf(
          `You have entered an invalid ${inputName} Address!`
        );
        if (index > -1) this.errors.splice(index, 1);
      }
    },
    checkSubmit() {
      var ipformat = /^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
      var ipaddr = this.ipAddress;
      var gateway = this.gateway;
      var subnet = this.subnet;
      var dns = this.dns;
      if (ipaddr.match(ipformat) == null) {
        let err = "You have entered an invalid IP Address!";
        if (!this.errors.includes(err)) this.errors.push(err);
      }
      if (gateway.match(ipformat) == null) {
        let err = "You have entered an invalid Gateway Address!";
        if (!this.errors.includes(err)) this.errors.push(err);
      }
      if (subnet.match(ipformat) == null) {
        let err = "You have entered an invalid SubnetMask Address!";
        if (!this.errors.includes(err)) this.errors.push(err);
      }
      if (dns.match(ipformat) == null) {
        let err = "You have entered an invalid DNS Address!";
        if (!this.errors.includes(err)) this.errors.push(err);
      }

      if (!this.errors.length) {
        return true;
      } else {
        return false;
      }
    },
    sendWifi() {
      if (this.checkSubmit()) {
        server
          .post(
            "/staticWifi",
            {
              ssid: this.ssid,
              password: this.password,
              ipAddress: this.ipAddress,
              gateway: this.gateway,
              subnet: this.netmaskToCidr(this.subnet),
              dns: this.dns,
            },
            {
              headers: {
                "Content-Type": "application/json;charset=UTF-8",
              },
            }
          )
          .then((res) => {
            if (res.status == 200) {
              this.errors = [];
              this.message = "Static WiFi settings have been saved.";
              this.addLog(`${this.getTime()} - Static WiFi settings have been saved.`);
              this.addLog(
                `${this.getTime()} - New SSID: ${this.ssid}
                ${this.getTime()} - New IP Address: ${this.ipAddress}`
              );
              this.ssid = "";
              this.password = "";
              this.ipAddress = "";
              this.gateway = "";
              this.subnet = "";
              this.dns = "";
              this.wifi = false;
              this.ethernet = false;
            } else {
              this.errors.push("Something went wrong. Please try again");
              this.addLog(
                `${this.getTime()} - Something went wrong while trying to save the new Static WiFi settings. Please try again`
              );
            }
          })
          .catch((error) => {
            this.addLog(
              `${this.getTime()} - Something went wrong while trying to save the new Static WiFi settings. Please try again`
            );
            console.log(error);
            this.message = "";
          });
      }
    },
    sendEthernet() {
      if (this.checkSubmit()) {
        server
          .post(
            "/staticEthernet",
            {
              ipAddress: this.ipAddress,
              gateway: this.gateway,
              subnet: this.netmaskToCidr(this.subnet),
              dns: this.dns,
            },
            {
              headers: {
                "Content-Type": "application/json;charset=UTF-8",
              },
            }
          )
          .then((res) => {
            if (res.status == 200) {
              this.errors = [];
              this.message = "Static Ethernet settings have been saved.";
              this.addLog(`${this.getTime()} - Static Ethernet settings have been saved.`);
              this.addLog(`${this.getTime()} - New IP Address: ${this.ipAddress}`);
              this.ipAddress = "";
              this.gateway = "";
              this.subnet = "";
              this.dns = "";
              this.wifi = false;
              this.ethernet = false;
            } else {
              this.errors.push("Something went wrong. Please try again");
              this.addLog(
                `${this.getTime()} - Something went wrong while trying to save the new Static Ethernet settings. Please try again`
              );
            }
          })
          .catch((error) => {
            this.addLog(
              `${this.getTime()} - Something went wrong while trying to save the new Static Ethernet settings. Please try again`
            );
            console.log(error);
            this.message = "";
          });
      }
    },
  },
  computed: {},
};
</script>

<style scoped>
/* DHCP and Static IP Pages - START */
.top_container {
  height: auto;
}
.mid_container .inner_container form {
  justify-content: flex-start;
}
.setting {
  color: #96ede5;
}
#wifiFields {
  width: 100%;
  flex-direction: column;
  align-items: center;
}
#ethernetFields {
  width: 100%;
  flex-direction: column;
  align-items: center;
}
.input_radio {
  margin: 20px 0;
  color: white;
  text-align: center;
}
.input_radio input {
  display: none;
}
.input_radio span {
  font-size: 20px;
  color: white;
}
.input_radio label {
  display: inline-block;
  font-size: 20px;
  font-weight: bold;
  line-height: 45px;
  height: 45px;
  width: 150px;
  background: rgba(0, 0, 0, 0.2);
  cursor: pointer;
}
.input_radio label:first-of-type {
  margin-right: 5px;
}
.input_radio label:hover,
.input_radio input[type="radio"]:checked + label {
  color: red;
  background-color: rgba(0, 0, 0, 0.5);
}
.input_row .wrong {
  color: #e11422;
  font-weight: bold;
}
.input_row .correct {
  color: #14e16d;
  font-weight: bold;
}

.message,
.errors {
  text-align: center;
  font-size: 1.2em;
  font-weight: bold;
  color: #14e16d;
  padding: 0.5em;
  margin: 0.5em;
}

.errors {
  color: #e11422;
}
</style>
