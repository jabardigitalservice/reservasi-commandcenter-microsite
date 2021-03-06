<template>
  <v-row>
    <v-col cols="12" class="mt-16">
      <h1 class="text-h5 text-md-h4 font-weight-medium mb-6 primary--text">Form Permohonan Reservasi <br>Kunjungan Jabar Command Center</h1>
      <h2 class="text-subtitle-1 darkgray--text mb-8 font-weight-medium">
        Harap mengisi data dibawah ini dengan sebenar-benarnya
      </h2>
      <v-form
        ref="form"
        v-on:submit.prevent="submitData()"
        v-model="valid"
        lazy-validation
      >
        <v-text-field
          v-model="name"
          :rules="this.$nameRules()"
          label="Nama Penanggung Jawab/Nama Pengunjung *"
          :maxLength="100"
          :counter="100"
          required
        ></v-text-field>
        <v-text-field
          v-model="nik"
          :rules="this.$nikRules()"
          label="Nomor Identitas (NIK) *"
          :counter="16"
          :maxLength="16"
          required
        ></v-text-field>
        <v-text-field
          v-model="organization"
          :rules="this.$organizationRules()"
          :maxLength="50"
          :counter="50"
          label="Nama Organisasi / Instansi"
        ></v-text-field>
        <v-text-field
          v-model="organizationAddress"
          :rules="this.$organizationAddressRules()"
          :counter="100"
          :maxLength="100"
          label="Alamat Organisasi / Instansi"
        ></v-text-field>
        <v-text-field
          v-model="phoneNumber"
          :rules="this.$phoneNumberRules()"
          :minLength="10"
          :maxLength="13"
          :counter="13"
          label="Nomor Telepon Penanggung Jawab *"
          required
        ></v-text-field>
        <v-text-field
          v-model="email"
          :rules="this.$emailRules()"
          :maxLength="50"
          :counter="50"
          label="Alamat Email *"
          required
        ></v-text-field>
        <v-textarea
          name="input-maksudTujuan"
          class="pb-2"
          v-model="purpose"
          :rules="this.$purposeRules()"
          :maxLength="255"
          :counter="255"
          label="Maksud dan Tujuan Kunjungan *"
          value=""
          hint="Isi Pesan"
        ></v-textarea>
        <v-menu
          ref="menu1"
          v-model="menu1"
          :close-on-content-click="false"
          transition="scale-transition"
          offset-y
          max-width="290px"
          min-width="auto"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="dateVisitorFormat"
              :rules="dateVisitorRules"
              label="Tanggal Kunjungan *"
              hint="Bulan/Hari/Tahun"
              persistent-hint
              prepend-icon="mdi-calendar"
              v-bind="attrs"
              @blur="dateVisitor = parseDate()"
              v-on="on"
              class="mb-8"
            ></v-text-field>
          </template>
          <v-date-picker
            v-model="dateVisitor"
            :min="minDates"
            :max="maxDates"
            :allowed-dates="allowedDates"
            @change="changeVisitors(timeVisitor, dateVisitor)"
            no-title
            @input="menu1 = false"
            class="mb-8"
          ></v-date-picker>
        </v-menu>
        <v-select
          v-model="timeVisitor"
          :rules="this.$timeVisitorRules()"
          :hint="
            timeVisitor != null
              ? `${timeVisitor.name}, Kapasitas ${timeVisitor.capacity} Orang`
              : ''
          "
          :items="itemsShift"
          item-text="time"
          item-value="id"
          label="Waktu Kunjungan"
          @change="changeVisitors(timeVisitor, dateVisitor)"
          persistent-hint
          return-object
          class="mb-8"
        ></v-select>
        <v-text-field
          v-model="visitors"
          :disabled="disableVisitors"
          :rules="visitorsRules"
          class="jumlah-peserta"
          ref="jumlahPeserta"
          type="number"
          :label="labelVisitor"
          suffix="Orang"
          :hint="avaibilityVisitor"
        ></v-text-field>
        <v-checkbox
          v-model="agree"
          :rules="this.$agreeRules()"
          required
        >
          <template v-slot:label>
            <div>
              Saya menyatakan bahwa saya menyetujui
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <a
                    href="#"
                    class="primary--text font-weight-medium text-decoration-none"
                    @click.stop = "openTermsAndConditions"
                    v-on="on"
                  >
                    syarat dan ketentuan
                  </a>
                </template>
                Lihat syarat dan ketentuan
              </v-tooltip>
              yang berlaku.
            </div>
          </template>
        </v-checkbox>
        <recaptcha
          class="mb-5"
          @error="onErrorCaptcha"
          @success="onSuccessCaptcha"
        />
        <v-alert dense outlined type="error" v-model="errorCaptcha">
          <strong>Captcha Wajib di isi !!</strong>
        </v-alert>
        <v-dialog
          persistent
          v-model="dialogSuccess"
          transition="dialog-top-transition"
          max-width="600"
        >
          <template>
            <v-card>
              <v-card-text>
                <v-col class="pt-7" align="center">
                  <v-icon class="success-dialog" x-large color="green darken-2"
                    >mdi-check-circle-outline</v-icon
                  >
                  <div class="text-h5 pa-5 green--text text--darken-2">
                    Terima kasih! Form permohonan reservasi kunjungan berhasil
                    terkirim.
                  </div>
                  <v-container>
                    <div class="text-h6">Kode Reservasi anda :</div>
                    <v-text-field
                      v-model="reservationCode"
                      class="pa-5"
                      ref="textToCopy"
                      :append-outer-icon="'mdi-content-copy'"
                      filled
                      clear-icon="mdi-close-circle"
                      readonly
                      type="text"
                      @click:append-outer="copyClipboard"
                    ></v-text-field>
                  </v-container>
                  <div class="text-h7 pa-1">
                    Silahkan mengecek kotak masuk / spam pada email anda untuk
                    melihat kembali
                    <b>Kode Reservasi.</b> Harap menyimpan
                    <b>Kode Reservasi</b> untuk mengetahui status permohonan
                    anda pada halaman
                    <nuxt-link
                      to="cek-reservasi"
                      class="text-decoration-none green--text text--darken-2"
                      >Cek Status Permohonan.</nuxt-link
                    >
                  </div>
                  <div class="text-h7 pa-1">
                    <v-btn
                      color="white--text green darken-2"
                      @click="closeDialog"
                      >Tutup</v-btn
                    >
                  </div>
                </v-col>
              </v-card-text>
              <v-card-actions class="justify-end"> </v-card-actions>
            </v-card>
          </template>
        </v-dialog>
        <v-btn
          block
          class="mb-3"
          color="success"
          type="submit"
          :loading="loading"
          :disabled="!valid"
        >
          Kirim Permohonan
        </v-btn>
        <nuxt-link to="/" class="btn-batalkan">
          <v-btn block color="error"> Batalkan </v-btn>
        </nuxt-link>
      </v-form>
    </v-col>
    <TermsAndConditions v-model="showTerms" />
  </v-row>
</template>
<script>
export default {
  data: (vm) => ({
    timeVisitor: {},
    loading: false,
    reservationCode: "#",
    labelVisitor: "Jumlah Peserta",
    disableVisitors: false,
    errorCaptcha: false,
    itemsShift: [],
    agree: false,
    name: "",
    nik: "",
    organization: "",
    organizationAddress: "",
    availabilityCount: Infinity,
    phoneNumber: "",
    email: "",
    purpose: "",
    dateVisitor: new Date().toISOString().substr(0, 10),
    dateVisitorRules: [(v) => !!v || "Tanggal Kunjungan wajib diisi"],
    dateVisitorFormat: vm.formatDate(new Date().toISOString().substr(0, 10)),
    menu1: false,
    visitors: "",
    visitorsRules: [],
    valid: false,
    recaptchaResponse: null,
    dialogSuccess: false,
    showTerms: false
  }),

  computed: {
    computedDateFormatted() {
      return this.formatDate(this.dateVisitor);
    },
    minDates() {
      return new Date().toISOString().substr(0, 10);
    },
    maxDates() {
      var getDate = new Date();
      var formatMaxDate = new Date(getDate.setMonth(getDate.getMonth() + 1))
        .toISOString()
        .substr(0, 10);
      return formatMaxDate;
    },
    avaibilityVisitor() {
      return `Kuota Peserta Sisa ${this.availabilityCount} Orang`;
    },
  },

  async mounted() {
    await this.getListShift();
  },

  watch: {
    dateVisitor() {
      this.dateVisitorFormat = this.formatDate(this.dateVisitor);
    },
  },

  methods: {
    visitorRuleNotEmpty: (v) => !!v || "Jumlah Peserta wajib diisi",
    visitorRuleNotFull(v) {
      return (
        (v && v <= this.availabilityCount) ||
        "Jumlah Peserta Maksimum " + this.availabilityCount + " Orang"
      );
    },
    visitorRuleLessThanZero: (v) =>
      (v && v > 0) || "Jumlah Peserta harus lebih dari nol",
    copyClipboard() {
      let textToCopy = this.$refs.textToCopy.$el.querySelector("input");
      textToCopy.select();
      document.execCommand("copy");
    },
    async getListShift() {
      const dataShift  = await this.$axios.$get("/command-center-shift?sortBy=time&orderDirection=asc&status=ACTIVE");
      this.itemsShift  = dataShift.data;
      this.timeVisitor = dataShift.data[0];
      this.changeVisitors(this.timeVisitor, this.dateVisitor);
    },
    allowedDates(val) {
      var dateNow = new Date();
      var dateLimitVisitor = new Date(dateNow.setMonth(dateNow.getMonth() + 1));

      var daysAllowed = [];

      for (
        var days = new Date();
        days <= dateLimitVisitor;
        days.setDate(days.getDate() + 1)
      ) {
        daysAllowed.push(new Date(days).toISOString().substr(0, 10));
      }

      return daysAllowed.indexOf(val) !== -1;
    },
    async changeVisitors(val, date) {
      this.visitors = "";
      this.$refs.form.resetValidation();
      this.$refs.jumlahPeserta.focus();
      let checkAvailibility = await this.$axios.get(
        `/command-center-availability?reservation_date=${date}&command_center_shift_id=${val.id}`
      );
      this.availabilityCount = checkAvailibility.data.data.available;
      if (this.availabilityCount <= 0) {
        this.visitorsRules.length = 0;
        this.labelVisitor = "Kuota Penuh";
        this.disableVisitors = true;
        this.visitorsRules.push(this.visitorRuleNotEmpty);
      } else {
        this.labelVisitor = "Jumlah Peserta";
        this.visitorsRules.length = 0;
        this.disableVisitors = false;
        this.visitorsRules.push(
          this.visitorRuleNotEmpty,
          this.visitorRuleLessThanZero,
          this.visitorRuleNotFull
        );
      }
    },
    formatDate(date) {
      if (!date) return null;

      const [year, month, day] = date.split("-");
      return `${month}/${day}/${year}`;
    },
    parseDate() {
      var date = new Date().toISOString().substr(0, 10);
      var dateFormatted = this.formatDate(date);
      const [month, day, year] = dateFormatted.split("/");
      return `${year}-${month.padStart(2, "0")}-${day.padStart(2, "0")}`;
    },
    onErrorCaptcha() {
      this.errorCaptcha = true;
    },
    async submitData() {
      if (this.$refs.form.validate()) {
        try {
          this.loading = true;
          let token = await this.$recaptcha.getResponse();
          await this.$axios
            .post(`/public/command-center-reservation`, {
              name: this.name,
              nik: this.nik,
              organization_name: this.organization,
              address: this.organizationAddress,
              phone_number: this.phoneNumber,
              email: this.email,
              purpose: this.purpose,
              visitors: this.visitors,
              reservation_date: this.dateVisitor,
              command_center_shift_id: this.timeVisitor.id,
            },
            {
              headers: {
                'recaptcha-token' : token,
              },
            },
            )
            .then((resp) => {
              this.dialogSuccess = true;
              this.reservationCode = resp.data.data.reservation_code;
            })
            .catch((e) => {
              this.loading = false;
              this.$toast.error(
                "Terjadi kesalahan silahkan menghubungi admin",
                {
                  duration: 5000,
                }
              );
            });
          await this.$recaptcha.reset();
        } catch (e) {
          this.loading = false;
        }
      }
    },
    onSuccessCaptcha(token) {
      this.recaptchaResponse = token;
    },
    closeDialog() {
      this.dialogSuccess = false;
      this.errorCaptcha = false;
      this.$router.push("/");
    },
    openTermsAndConditions() {
      this.showTerms = true;
    }
  },
};
</script>
