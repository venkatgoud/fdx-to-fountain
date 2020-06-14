<template>
  <div class="flex items-center flex-col pb-2">
    <h1 class="w-64 mx-auto my-4 text-lg">Finaldraft to Fountain converter</h1>
    <label
      class="w-64 flex flex-col items-center px-2 py-3 bg-white text-blue-600 rounded-lg shadow-lg tracking-wide uppercase border border-blue-400 cursor-pointer hover:bg-blue-600 hover:text-white"
    >
      <svg
        class="w-6 h-4"
        fill="currentColor"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 20 20"
      >
        <path
          d="M16.88 9.1A4 4 0 0 1 16 17H5a5 5 0 0 1-1-9.9V7a3 3 0 0 1 4.52-2.59A4.98 4.98 0 0 1 17 8c0 .38-.04.74-.12 1.1zM11 11h3l-4-4-4 4h3v3h2v-3z"
        />
      </svg>
      <span class="mt-1 text-base leading-normal">Load screenplay</span>
      <input type="file" class="hidden" name="inputfile" ref="inputFiles" @change="onFileChange" />      
    </label>
    <p class="text-red-600 py-2" v-show="error"> {{error}} </p>

    <div class="flex flex-col px-4 my-4" v-if="isFileLoaded">
      <div class="self-end py-4">
        <button
          v-on:click="download"
          class="border border-blue-400 hover:bg-blue-600 hover:text-white text-blue-600 py-2 px-4 rounded inline-flex items-center"
        >
          <svg class="w-4 h-4 mr-2" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
            <path d="M13 8V2H7v6H2l8 8 8-8h-5zM0 18h20v2H0v-2z" />
          </svg>
          <span>Download</span>
        </button>
      </div>
      <div
        style="width: 210mm;"
        class="flex flex-col mx-auto text-sm border-l-2 border-r-2 border-gray-400"
      >
        <template v-for="(token, index) in tokens">
          <br :key="index" v-if="token.type === 'dialogue_begin'" />
          <p :key="index" v-else :class="cssClass(token)">{{token.text}}</p>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { saveAs } from "file-saver";
import { finalDraftToFountain } from "screenplay-parsers";
import { parse } from "fountain-parser";

function constructFileName(file) {
  return file + ".fountain";
}
export default {
  name: "App",
  methods: {
    cssClass: function(token) {
      switch (token.type) {
        case "scene_heading":           
          return "font-semibold underline px-10";
        case "action":
          return "w-full py-1 px-10";
        case "character":
          return "self-center leading-3";
        case "dialogue":
          return "w-7/12 self-center px-20 leading-2";
        case "parenthetical":
          return "w-7/12 self-center px-20 leading-3";
        case "transition":
          return "self-end";
        default:
          break;
      }
    },
    download: function() {
      var blob = new Blob([this.content], {
        type: "text/plain;charset=utf-8"
      });
      var fountainFileName = constructFileName(this.fileName);
      saveAs(blob, fountainFileName);
    },
    onFileChange: function() {
      this.error = "";
      if (this.$refs.inputFiles.files.length > 0) {
        const file = this.$refs.inputFiles.files[0];
        this.fileName = file.name;
        var fileReader = new FileReader();
        const _instance = this;
        fileReader.onload = function(evt) {
          _instance.isFileLoaded = true;
          finalDraftToFountain(evt.target.result)
            .then(data => {
              _instance.content = data;

              parse(_instance.content, output => {
                _instance.tokens = output.tokens;
              });
            })
            .catch(error => {
              _instance.error = error;
              _instance.isFileLoaded = false;
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
      error: "",
      content: "",
      tokens: []
    };
  }
};
</script>