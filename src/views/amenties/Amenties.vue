<template>
  <div>
    <CRow>
      <CCol col="12">
        <CButton
          color="info"
          class="mx-auto d-block mb-4"
          @click="showAddModal = true"
        >
          <CIcon name="cilPlus" />
        </CButton>
        <CModal
          title="Qo'shish"
          color="info"
          :show.sync="showAddModal"
          size="lg"
          :centered="true"
        >
          <CRow>
            <CCol col="12">
              <CInput
                label="qulaylik nomi"
                v-model="title"
                valid-feedback="Yaxshi."
                invalid-feedback="Kamida bitta so'z kiriting."
                :is-valid="validator"
                required
              />
            </CCol>
            <CCol col="12">
              <editor
                api-key="72yygutms5bezwcxavaanlhe1atalva8wzzssnbsnqbbifhh"
                v-model="description"
                :init="$store.state.config"
              />
              <div class="custom-file my-3">
                <input
                  ref="inputVal"
                  @change="fileUpload"
                  type="file"
                  class="custom-file-input"
                  id="validatedCustomFile"
                  required
                />
                <label
                  ref="image"
                  class="custom-file-label"
                  for="validatedCustomFile"
                  >Rasm yuklang...</label
                >
              </div>
            </CCol>
          </CRow>
          <div slot="footer">
            <div slot="footer-wrapper">
              <CButton
                color="success"
                class="float-right"
                @click="add()"
                :disabled="loader"
              >
                <CSpinner v-if="loader" color="warning" size="sm" />

                <span v-else>Saqlash</span>
              </CButton>
            </div>
          </div>
        </CModal>
      </CCol>
    </CRow>
    <Table
      :button="bshow"
      :theads="thead"
      :tbodies="amenties"
      :objects="obj"
      @getItemEdit="getEdit"
    />

    <CModal
      title="O'zgartirish"
      color="info"
      :show.sync="showAddModaledit"
      size="lg"
      :centered="true"
    >
      <CRow>
        <CCol col="12">
          <CInput
            label="qulaylik nomi"
            v-model="amenty.title"
            valid-feedback="Yaxshi."
            invalid-feedback="Kamida bitta so'z kiriting."
            :is-valid="validator"
            required
          />
        </CCol>
        <CCol col="12">
          <editor
            api-key="72yygutms5bezwcxavaanlhe1atalva8wzzssnbsnqbbifhh"
            v-model="amenty.description"
            :init="$store.state.config"
          />
          <div class="custom-file my-3">
            <input
              ref="inputVal"
              @change="uploadImage"
              type="file"
              class="custom-file-input"
              id="validatedCustomFile"
              required
            />
            <label
              ref="aImage"
              class="custom-file-label"
              for="validatedCustomFile"
              >Rasm yuklang...</label
            >
          </div>
        </CCol>
      </CRow>
      <div slot="footer">
        <div slot="footer-wrapper">
          <CButton
            color="success"
            class="float-right"
            @click="update()"
            :disabled="loader"
          >
            <CSpinner v-if="loader" color="warning" size="sm" />
            <span v-else>Saqlash</span>
          </CButton>
        </div>
      </div>
    </CModal>
  </div>
</template>

<script>
// import Editor from "@tinymce/tinymce-vue";
import { mapGetters } from "vuex";
import Table from "@/components/Table";
import axios from "axios";
import Editor from "@tinymce/tinymce-vue";
import Loader from "../../components/Loader";

export default {
  components: {
    Table,
    editor: Editor,
    Loader,
  },
  data() {
    return {
      loader: false,
      amenty: [],
      bshow: { warning: true, delete: true },
      imgSet: false,
      imagevalidation: true,
      addimagevalidation: false,
      showAddModal: false,
      showAddModaledit: false,
      title: "",
      description: "",
      form: "",
      thead: ["Title", "Icon", "Description", "Amallar"],
      obj: ["title", "icon", "description"],
    };
  },
  methods: {
    validator(val) {
      return (val && val.length) > 0 ? true : false;
    },
    fileUpload(event) {
      this.form = new FormData();
      this.form.append("title", this.title);
      this.form.append("description", this.description);
      this.form.append("icon", event.target.files[0]);
      this.$refs.image.innerHTML = event.target.files[0].name;
    },

    getEdit(val) {
      this.showAddModaledit = true;
      this.itemId = val;
      this.$store
        .dispatch("edit", `${this.$store.state.currentItemUrlApi}/${val}/`)
        .then(() => {
          this.amenty = this.$store.state.singleItemEdit;
          console.log(this.amenty);
        });
    },
    async add() {
      await axios
        .post("amenity/admin/", this.form)
        .then((res) => {
          this.$toast.success("Muvaffaqiyatli bajarildi.");
          console.log(res);
        })
        .catch(() => {
          this.$toast.error("xatolik yuz  berdi.");
        });
    },
    uploadImage(event) {
      this.$refs.aImage.innerHTML = event.target.files[0].name;
      if (
        event.target.files[0] &&
        (event.target.files[0].type == "image/png" ||
          event.target.files[0].type == "image/jpeg" ||
          event.target.files[0].type == "image/jpg")
      ) {
        this.amenty.icon = event.target.files[0];
      } else {
        this.$toast.error(
          "Faqat jpeg,png yoki jpg formatda faylni yuklang va 5MB ko'p bo'lmasligi kerak."
        );
      }
    },
    async update() {
      this.loader = true;
      let form = new FormData();
      console.log(this.amenty);
      form.append("title", this.amenty.title);
      form.append("description", this.amenty.description);
      // form.append("icon", this.amenty.icon);
      await axios
        .patch(`amenity/admin/${this.itemId}/`, form)
        .then((res) => {
          this.$toast.success("Muvaffaqiyatli bajarildi.");
          console.log(res);
        })
        .catch(() => {
          this.$toast.error("xatolik yuz  berdi.");
        });
    },
  },
  computed: {
    ...mapGetters(["amenties"]),
  },
  mounted() {
    window.addEventListener("load", function() {
      this.loader = false;
    });
  },
  created() {
      this.$store.commit('setShowLoader',true);
    this.$store.dispatch("getamenties")
     .then(() => {
            this.$store.commit('setShowLoader',false);
        })
  },
};
</script>
