<script setup>
// import PostInput from "@/components/PostInput.vue";
import PostPreview from "@/components/PostPreview.vue";
import { useMemberStore } from "../stores/member";
import axios from "axios";
import FileUploader from "./FileUploader.vue";
</script>
<script>
export default {
  props: {
    no: Number,
    type: String,
  },
  data() {
    return {
      form: {
        title: "",
        content: "",
        attached: [],
      },
      uploadProgress: 0,
      fileUploaderNextKey: 1,
      fileUploaderMap: {},
    };
  },
  async created() {
    if (this.GETURI === undefined) return;
    const result = await axios.get(this.GETURI);
    this.form.title = result.data.title;
    this.form.content = result.data.content;
    this.form.attached = result.data.attached.map((item) => item.id);
    this.form.attached.forEach((id) => {
      this.fileUploaderMap[this.fileUploaderNextKey++] = id;
    });
  },
  computed: {
    createNormalPost() {
      return typeof this.no === "number" && !isNaN(this.no);
    },
    method() {
      switch (this.type) {
        case "about":
        case "rules":
          return axios.put;
        case "notices":
          return this.createNormalPost ? axios.put : axios.post;
        default:
          throw "unknown type: " + this.type;
      }
    },
    GETURI() {
      switch (this.type) {
        case "about":
        case "rules":
          return this.type;
        case "notices":
          return this.createNormalPost ? `notices/${this.no}` : undefined;
        default:
          throw "unknown type: " + this.type;
      }
    },
    WriteURI() {
      switch (this.type) {
        case "about":
        case "rules":
          return this.type;
        case "notices":
          return this.createNormalPost ? `notices/${this.no}` : "notices";
        default:
          throw "unknown type: " + this.type;
      }
    },
  },
  methods: {
    routeToReturn(no) {
      switch (this.type) {
        case "about":
          return {
            name: "about",
          };
        case "rules":
          return {
            name: "rules",
          };
        case "notices":
          return {
            name: "notice",
            params: {
              no: no,
            },
          };
        default:
          throw "unknown type: " + this.type;
      }
    },
    fileUploaded(event) {
      this.form.attached.push(event.id);
    },
    fileRemoved(event) {
      this.form.attached = this.form.attached.filter(
        (item) => item !== event.id
      );
    },
    async submit() {
      try {
        const store = useMemberStore();
        const result = await this.method(this.WriteURI, this.form, {
          headers: {
            Authorization: store.authorizationHeader,
          },
        });
        this.$router.push(this.routeToReturn(result.data.no));
      } catch (error) {
        console.error(error);
      }
    },
    newUploader() {
      this.fileUploaderMap[this.fileUploaderNextKey++] = "";
    },
    async uploadImageInsidePost(event) {
      const formData = new FormData();
      formData.append("uploaded", event.target.files[0]);
      const response = await axios.post("uploaded", formData, {
        headers: {
          Authorization: useMemberStore().authorizationHeader,
        },
      });
      this.form.content += `![????????? ????????????](${axios.defaults.baseURL}uploaded/${response.data.id})`;
    },
  },
};
</script>
<template>
  <div class="container-fluid">
    <div class="row">
      <form class="col" @submit.prevent="submit">
        <div class="form-floating mb-1">
          <input
            type="text"
            class="form-control"
            id="title"
            v-model="form.title"
            placeholder="??????"
            required
          />
          <label for="title">??????</label>
        </div>
        <div class="function-bar">
          <i class="bi-type-h1" @click="form.content += '\n# ??????1'"></i>
          <i class="bi-type-h2" @click="form.content += '\n## ??????2'"></i>
          <i class="bi-type-h3" @click="form.content += '\n### ??????3'"></i>
          <i class="bi-type-italic" @click="form.content += '*?????????*'"></i>
          <i
            class="bi-type-strikethrough"
            @click="form.content += '~~?????????~~'"
          ></i>
          <i class="bi-type-bold" @click="form.content += '**??????**'"></i>
          <label for="image-inside-post">
            <i class="bi-card-image"></i>
          </label>
          <input
            type="file"
            id="image-inside-post"
            @change="uploadImageInsidePost"
            accept="image/*"
            hidden
          />
          <i
            class="bi-link"
            @click="form.content += '[????????????](http://yonseimunhak.com)'"
          ></i>
        </div>
        <div class="form-floating mb-1">
          <textarea
            class="form-control"
            id="content"
            v-model="form.content"
            placeholder="??????"
            required
          ></textarea>
          <label for="content">??????</label>
        </div>
        <div>
          <label for="form-file-multiple"><small>????????????</small></label>
          <FileUploader
            @upload="fileUploaded"
            @uploadedRemove="fileRemoved"
            v-for="[key, id] of Object.entries(fileUploaderMap)"
            :key="key"
            :id="id"
            class="uploader"
          ></FileUploader>
          <button type="button" class="btn btn-light mt-1" @click="newUploader">
            ???????????? ??????
          </button>
        </div>
        <button type="submit" class="btn btn-light mt-1 d-none d-lg-block">
          ??????
        </button>
      </form>
      <PostPreview :form="form" class="col d-none d-lg-block"></PostPreview>
    </div>
    <PostPreview :form="form" class="row d-lg-none"></PostPreview>
    <button type="button" class="btn btn-light mt-1 d-lg-none" @click="submit">
      ??????
    </button>
  </div>
</template>
<style scoped>
textarea#content {
  min-height: 50vh;
}
.function-bar i {
  font-size: 1.5rem;
  margin: auto 5px auto 5px;
  cursor: pointer;
}
</style>
