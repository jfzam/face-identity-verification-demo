<template>
  <div class="container pt-6">
    <div class="modal">
      <div class="modal-background"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">{{modalTitle}}</p>
        </header>
        <section class="modal-card-body">
          <div class="columns is-centered">
            <figure class="image is-128x128">
              <img id="faceimage" src="../assets/verified.jpg" />
            </figure>
          </div>
          <br />
          <div class="columns is-centered">
            <h1 class="title">{{modalBody}}</h1>
          </div>
          <br />
          <br />
        </section>
      </div>
    </div>

    <div class="columns is-centered">
      <div class="column is-half">
        <div class="card" v-if="isIdValid == false">
          <header class="card-header">
            <p class="card-header-title">User Information</p>
          </header>
          <div class="card-content">
            <div class="content">
              <div class="field">
                <div class="control">
                  <input
                    class="input is-primary"
                    type="text"
                    placeholder="First Name"
                    v-model="firstname"
                  />
                </div>
              </div>
              <div class="field">
                <div class="control">
                  <input
                    class="input is-primary"
                    type="text"
                    placeholder="Middle Name"
                    v-model="middlename"
                  />
                </div>
              </div>
              <div class="field">
                <div class="control">
                  <input
                    class="input is-primary"
                    type="text"
                    placeholder="Last Name"
                    v-model="lastname"
                  />
                </div>
              </div>
              <div class="field">
                <div class="control">
                  <input
                    class="input is-primary"
                    type="date"
                    placeholder="birth Date"
                    v-model="birthdate"
                  />
                </div>
              </div>
              <div class="field">
                <span class="has-text-danger">{{errorMessage}}</span>
              </div>
              <div class="field">
                <div class="file is-primary is-right">
                  <label class="file-label">
                    <input class="file-input" type="file" name="resume" v-on:change="uploadId" />
                    <span class="file-cta">
                      <span class="file-icon">
                        <i class="fas fa-upload"></i>
                      </span>
                      <span class="file-label">Upload ID</span>
                    </span>
                  </label>
                </div>
              </div>
              <div>
                <figure class="image">
                  <img id="idcard" />
                </figure>
              </div>
              <br />
              <div v-if="isLoading" class="field">
                <progress class="progress is-small is-primary" max="100"></progress>
              </div>
              <div class="field">
                <button
                  id="verify"
                  class="button is-link is-fullwidth"
                  v-on:click="onVerify"
                  v-if="texTractName!=''"
                >{{verifyLabel}}</button>
              </div>
            </div>
          </div>
        </div>
        <br />
        <div class="card" v-if="isIdValid">
          <header class="card-header">
            <p class="card-header-title">Face Verification</p>
          </header>
          <div class="card-content">
            <div class="content">
              <div class="field">
                <div class="file is-primary is-right">
                  <label class="file-label">
                    <input class="file-input" type="file" name="resume" v-on:change="uploadImage" />
                    <span class="file-cta">
                      <span class="file-icon">
                        <i class="fas fa-upload"></i>
                      </span>
                      <span class="file-label">Capture</span>
                    </span>
                  </label>
                </div>
              </div>
              <div>
                <figure class="image">
                  <img id="imageself" />
                </figure>
              </div>
              <div class="field">
                <span class="has-text-danger">{{errorMessage1}}</span>
              </div>
              <div class="field" v-if="hasFaceImage">
                <button
                  id="compare"
                  class="button is-link is-fullwidth"
                  v-on:click="onCompare"
                >Submit</button>
              </div>
              <br />
              <div v-if="isLoading1" class="field">
                <progress class="progress is-small is-primary" max="100"></progress>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

const path = "http://localhost:8888/";
let validId = null;
let faceImage = null;

export default {
  name: "Form",
  data() {
    return {
      firstname: "",
      middlename: "",
      lastname: "",
      birthdate: null,
      errorMessage: "",
      errorMessage1: "",
      isIdValid: false,
      isLoading: false,
      isLoading1: false,
      texTractName: "",
      texTractDate: "",
      verifyLabel: "Verify",
      modalTitle: "Identity Verification",
      modalBody: "Your ID is Verified",
      hasFaceImage: false
    };
  },
  methods: {
    testOnClick() {
      var modal = document.querySelector(".modal"); // assuming you have only 1
      var html = document.querySelector("html");
      modal.classList.add("is-active");
      html.classList.add("is-clipped");

      modal
        .querySelector(".modal-background")
        .addEventListener("click", function(e) {
          e.preventDefault();
          modal.classList.remove("is-active");
          html.classList.remove("is-clipped");
        });
    },
    uploadId(event) {
      var image = event.target.files[0];
      validId = image;
      var imageElement = document.getElementById("idcard");
      imageElement.src = URL.createObjectURL(image);

      var bodyFormData = new FormData();
      bodyFormData.append("image", image);

      var headers = new Headers();
      headers.append("Content-Type", "multipart/form-data");
      headers.append("Access-Control-Allow-Origin", "http://localhost:8080");
      headers.append("token", "c7a69b46-a490-4a64-a503-1e5971824a3d");
      let self = this;
      self.isLoading = true;
      axios({
        method: "post",
        url: path + "api/analyze",
        data: bodyFormData,
        headers: headers
      })
        .then(function(response) {
          //handle success
          console.log(response.data.result);
          var result = response.data.result;
          var idFullName = String(result["Last Name First Name. Middle Name"]);
          var idBirthDate = String(result["Date of Birth"]);
          
          self.texTractName = "".concat(
            idFullName
              .toUpperCase()
              .replace(",", "")
              .split(" ")
              .sort()
          );
          self.texTractDate = idBirthDate;
          self.isLoading = false;
        })
        .catch(function(response) {
          //handle error
          console.log(response);
          self.isLoading = false;
        });
    },
    onVerify() {
      let self = this;

      var nameInput = "".concat(
        (
          self.firstname.toUpperCase() +
          " " +
          self.middlename.toUpperCase() +
          " " +
          self.lastname.toUpperCase()
        )
          .split(" ")
          .sort()
      );

      var bdate = self.texTractDate.split('/').join('-')

      if (!self.texTractName) {
        self.errorMessage = "* No name Found on ID";
      } else if (self.texTractName != nameInput) {
        self.errorMessage = "* Names doesn't Match";
      } else if (bdate != self.birthdate){
        self.errorMessage = "* Birthdates doesn't Match";
      }  else {
        self.errorMessage = "";
        self.isIdValid = true;
        self.verifyLabel = "Verified";
        document.getElementById("verify").className =
          "button is-primary is-fullwidth is-outlined";
        var modal = document.querySelector(".modal"); // assuming you have only 1
        var html = document.querySelector("html");
        modal.classList.add("is-active");
        html.classList.add("is-clipped");

        modal
          .querySelector(".modal-background")
          .addEventListener("click", function(e) {
            e.preventDefault();
            modal.classList.remove("is-active");
            html.classList.remove("is-clipped");
          });
      }
    },
    uploadImage(event) {
      var image = event.target.files[0];
      faceImage = image;
      var imageElement = document.getElementById("imageself");
      imageElement.src = URL.createObjectURL(event.target.files[0]);
      console.log(faceImage);
      this.hasFaceImage = true;
    },
    onCompare() {
      let self = this;
      this.isLoading1 = true;
      var bodyFormData = new FormData();
      bodyFormData.append("image", validId);
      bodyFormData.append("image", faceImage);

      var headers = new Headers();
      headers.append("Content-Type", "multipart/form-data");
      headers.append("Access-Control-Allow-Origin", "http://localhost:8080");
      headers.append("token", "c7a69b46-a490-4a64-a503-1e5971824a3d");
      //let self = this;
      axios({
        method: "post",
        url: path + "api/compare",
        data: bodyFormData,
        headers: headers
      })
        .then(function(response) {
          //handle success
          console.log(response.data.result);
          if (response.data.result.isMatch) {
            self.modalTitle = "Face Recognition";
            self.modalBody = "Face Matched Successfully";
             self.errorMessage1 = "";
            var modal = document.querySelector(".modal"); // assuming you have only 1
            var html = document.querySelector("html");
            modal.classList.add("is-active");
            html.classList.add("is-clipped");

            modal
              .querySelector(".modal-background")
              .addEventListener("click", function(e) {
                e.preventDefault();
                modal.classList.remove("is-active");
                html.classList.remove("is-clipped");
              });
          } else {
            self.errorMessage1 = "* Face and Id Picture doesn't match";
          }

          self.isLoading1 = false;
        })
        .catch(function(response) {
          //handle error
          console.log(response);
          self.errorMessage1 = "* Face and Id picture doesn't match";
          self.isLoading1 = false;
        });
    }
  }
};
</script>