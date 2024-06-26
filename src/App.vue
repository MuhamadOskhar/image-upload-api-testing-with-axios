<template>
  <v-container>
    <v-row class="d-flex justify-center mt-5 pt-5" style="min-height: 100vh">
      <v-col cols="12" md="6" lg="5">
        <v-card title="Upload Area">
          <v-card-text class="d-flex flex-column gap-4">
            <v-text-field
              v-model="form.endpoint"
              label="Endpoint"
              placeholder="http[s]://[yourSite]/[yourEndPoint]"
              required
            ></v-text-field>

            <v-text-field
              v-model="form.parameter_name"
              label="Name of Parameter"
              placeholder="Specific name for parameter required by your API endpoint"
              required
            ></v-text-field>

            <!-- Method Select -->
            <v-select
              v-model="form.method"
              :items="methods"
              label="Method"
              placeholder="Select HTTP Method"
              required
            ></v-select>

            <!-- Auth Token Checkbox -->
            <v-checkbox
              v-model="form.use_auth_token"
              label="Use Auth Token"
              hide-details
            ></v-checkbox>

            <v-text-field
              v-if="form.use_auth_token"
              v-model="form.auth_token"
              label="Auth token"
              placeholder="Bearer [your token]"
              required
            ></v-text-field>

            <!-- Avatar and Upload -->
            <div class="d-flex gap-4">
              <!-- 👉 Avatar -->
              <v-avatar
                rounded="lg"
                size="100"
                class="me-6"
                :src="data.profile_picture"
              />

              <!-- 👉 Upload Photo -->
              <div class="d-flex flex-column justify-center gap-2">
                <div class="d-flex flex-wrap gap-2">
                  <v-btn color="primary" @click="uploadPicture">
                    <v-icon icon="mdi-cloud-upload" class="d-sm-none" />
                    <span class="d-none d-sm-block">Send your picture</span>
                  </v-btn>

                  <input
                    id="gambar"
                    ref="refInputEl"
                    type="file"
                    name="file"
                    accept=".jpeg,.png,.jpg,GIF"
                    hidden
                    @input="sendPicture"
                  />
                </div>

                <p class="mb-0 gray-text">Only jpg, jpeg, png.</p>
              </div>
            </div>
          </v-card-text>
        </v-card>
      </v-col>

      <v-col cols="12" md="6" lg="5">
        <!-- Log Area -->
        <v-card title="Error Log Area" color="red" dark>
          <v-card-text v-if="logMessage"
            >{{ logMessageStatus }}<br /><br />{{ logMessage }}</v-card-text
          >
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import {
  VRow,
  VCol,
  VCard,
  VCardText,
  VAvatar,
  VBtn,
  VIcon,
  VTextField,
  VSelect,
} from "vuetify/components";
import axios from "axios";
import Swal from "sweetalert2";

export default {
  name: "UploadAvatar",
  components: {
    VRow,
    VCol,
    VCard,
    VCardText,
    VAvatar,
    VBtn,
    VIcon,
    VTextField,
    VSelect,
  },
  data() {
    return {
      data: {
        profile_picture: "", // Placeholder for initial image URL
      },
      form: {
        endpoint: "",
        parameter_name: "foto_profile",
        use_auth_token: true,
        auth_token: "",
        method: "POST",
      },
      methods: ["POST", "PUT"],
      refInputEl: null,
      logMessage: "",
      logMessageStatus: "",
    };
  },
  methods: {
    uploadPicture() {
      this.$refs.refInputEl.click();
    },
    async sendPicture(file) {
      const files = file.target.files[0];
      if (files) {
        const fileReader = new FileReader();
        // Validasi tipe file sebelum menampilkan gambarnya
        if (
          files.type === "image/jpeg" ||
          files.type === "image/png" ||
          files.type === "image/jpg"
        ) {
          fileReader.readAsDataURL(files);
          fileReader.onload = async () => {
            try {
              this.data.profile_picture = fileReader.result;

              const formData = new FormData();
              formData.append(this.form.parameter_name, fileReader.result);

              const response = await axios({
                method: this.form.method.toLowerCase(),
                url: this.form.endpoint,
                data: formData,
                headers: this.form.use_auth_token
                  ? { Authorization: this.form.auth_token }
                  : {},
              });

              if (response.data.success) {
                await Swal.fire({
                  toast: true,
                  position: "top",
                  iconColor: "white",
                  color: "white",
                  background: "green",
                  showConfirmButton: false,
                  timerProgressBar: true,
                  timer: 1500,
                  icon: "success",
                  title: response.data.success.message,
                });
              }
            } catch (error) {
              console.log(error);

              Swal.fire({
                toast: true,
                position: "top",
                iconColor: "white",
                color: "white",
                background: "red",
                showConfirmButton: false,
                timerProgressBar: true,
                timer: 1500,
                icon: "error",
                title: "terjadi kesalahan saat upload gambar!",
              });

              this.logMessageStatus = "Error status: " + error.response.status;
              this.logMessage =
                "Response from server: " + JSON.stringify(error.response.data);
            }
          };
        }
      }
    },
  },
};
</script>

<style scoped>
.d-flex {
  display: flex;
}
.d-sm-none {
  display: none;
}
.d-none {
  display: none;
}
.d-sm-block {
  display: block;
}
.me-6 {
  margin-right: 1.5rem;
}
.gray-text {
  color: gray;
}
</style>
