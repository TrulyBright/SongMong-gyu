<script setup>
import axios from "axios";
</script>
<script>
export default {
  props: {
    id: String,
  },
  data() {
    return {
      file: {
        id: "",
        name: "",
        contentType: "",
      },
    };
  },
  async created() {
    const response = await axios.get("uploaded/" + this.id + '/info');
    this.file.id = response.data.id;
    this.file.name = response.data.name;
    this.file.contentType = response.data.content_type;
  },
};
</script>
<template>
  <div>
    <a :href="axios.uploadedFilePath(file.id)" download>
      <small
        ><i
          :class="
            'bi-filetype-' +
            file.name.split('.')[file.name.split('.').length - 1].toLowerCase()
          "
        ></i
        >{{ file.name }}</small
      ></a
    >
  </div>
</template>
<style scoped></style>
