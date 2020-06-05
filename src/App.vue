<template>
  <div>
    <section class="section has-background-grey-lighter">
      <div class="container">
        <h1 class="subtitle">Finaldraft to Fountain</h1>
        <div class="field">
          <div id="input-file" class="file has-name is-info">
            <label class="file-label">
              <input
                class="file-input"
                type="file"
                name="inputfile"
                ref="inputFiles"
                @change="onFileChange"
              />
              <span class="file-cta">
                <span class="file-icon">
                  <i class="fas fa-upload"></i>
                </span>
                <span class="file-label">Choose a file…</span>
              </span>
              <span class="file-name">{{fileName}}</span>
            </label>
          </div>
        </div>
      </div>
    </section>
    <section class="section has-background-grey" style="padding: 1rem 1.5rem;" v-if="isFileLoaded">
      <div class="navbar has-background-grey">
        <div class="navbar-menu">
          <div class="navbar-end">
            <a class="button is-dark navbar-item" v-on:click="download">
              <strong>Download</strong>
            </a>
          </div>
        </div>
      </div>
      <div class="container is-fluid">
        <div class="notification" style="padding: 2rem 15rem;" v-html="displayContent"></div>
      </div>
    </section>
  </div>
</template>

<script>
import { saveAs } from "file-saver";
import { finalDraftToFountain } from "screenplay-parsers";
function constructFileName(file) {
  return file + ".fountain";
}
export default {
  name: "App",
  methods: {
    download: function() {
      var blob = new Blob([this.content], {
        type: "text/plain;charset=utf-8"
      });
      var fountainFileName = constructFileName(this.fileName);
      saveAs(blob, fountainFileName);
    },
    onFileChange: function() {
      if (this.$refs.inputFiles.files.length > 0) {
        const file = this.$refs.inputFiles.files[0];
        this.fileName = file.name;
        var fileReader = new FileReader();
        const _instance = this;
        fileReader.onload = function(evt) {
          _instance.isFileLoaded = true;
          finalDraftToFountain(evt.target.result)
            .then(data => {
              _instance.displayContent = data.replace(/\n/g, "<br/>");
              _instance.content = data;
            })
            .catch(error => {
              _instance.displayContent = error;
            });
        };
        fileReader.readAsText(file);
      }
    }
  },
  data: function() {
    return {
      isFileLoaded: false,
      fileName: "No file chosen",
      displayContent: "",
      content: ""
    };
  }
};
</script>