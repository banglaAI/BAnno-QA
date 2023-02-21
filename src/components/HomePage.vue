<template>
  <div class="HomePage container mt-3 mb-3">
    <div v-if="fileUploaded">
      <AnnotationsPage v-bind:json="json" v-bind:file="file"/>
    </div>
    <div v-else-if="customArticle">
      <AnnotateArticle />
    </div>
    <div v-else>
      <h1>BAnno QA: An annotation tool for constructing Bengali Question Answering Dataset </h1>

      <p>উইকিপিডিয়া নিবন্ধ টীকা করতে এখানে ক্লিক করুন: </p>
      <b-button :size="''" :variant="'success'" v-on:click="addArticles()">শুরু</b-button>
       <br><br>
      <br>
      ফাইল আপলোড করতে এবং টীকা চালিয়ে যেতে, প্রথমে ফাইলটি ব্রাউজ করুন:
      <br>
      <div class="uploadBar">
        <b-form-file
          v-model="file"
          :state="Boolean(file)"
          placeholder="একটি JSON ফাইল আপলোড করুন"
          accept=".json"
        ></b-form-file>
      </div>
      <br>
      <b-button :size="''" :variant="'primary'" v-on:click="readFile()">আপলোড</b-button>
    </div>
    </div>
</template>

<script>
import AnnotationsPage from "./AnnotationsPage.vue";
import AnnotateArticle from "./AnnotateArticle";

export default {
  name: "HomePage",
  data: function() {
    return {
      jsonData: {
        data: [
          {
            title: "Question answering",
            paragraphs: [
              {
                context:
                  "Question answering (QA) is a computer science discipline within the fields of information retrieval and natural language processing (NLP), which is concerned with building systems that automatically answer questions posed by humans in a natural language.",
                qas: []
              },
              {
                context:
                  "A QA implementation, usually a computer program, may construct its answers by querying a structured database of knowledge or information, usually a knowledge base. More commonly, QA systems can pull answers from an unstructured collection of natural language documents.",
                qas: []
              }
            ]
          },
          {
            title: "Natural language processing",
            paragraphs: "[...]"
          }
        ]
      },
      fileUploaded: false,
      file: null,
      json: null,
      customArticle: false
    };
  },
  methods: {
    readFile: function() {
      var reader = new FileReader();
      reader.onload = function(event) {
        this.json = JSON.parse(event.target.result);
        this.fileUploaded = true;
      }.bind(this);
      reader.readAsText(this.file);
    },
    addArticles: function () {
      this.customArticle = true;
    },
    readFile2: function() {
      var reader = new FileReader();
      reader.onload = function(event) {
        let tempdata = JSON.parse(event.target.result);
        let alldata = [];
        for (var i = 0; i < tempdata.length; i++) {
          if (tempdata[i]!=="") {
              alldata.push({
                title: "N/A",
                paragraphs: [{
                  context: tempdata[i],
                  qas: []
                }]
            });
          }
        }
        this.json = {
          data: alldata,
        };
        this.fileUploaded = true;
      }.bind(this);
      reader.readAsText(this.file);
    },
    readFile3: function() {
      var reader = new FileReader();
      reader.onload = function(event) {
        let tempdata = JSON.parse(event.target.result);
        this.json = {data: []};
        for (var i = 0; i < tempdata.length; i++) {
          let paras = [];
          for (var j = 0; j<tempdata[i].length; j++){
            if (tempdata[i][j]!=="") {
              paras.push({
                  context: tempdata[i][j],
                  qas: []
                });
            }
          }
          this.json.data.push({
            title: "N/A",
            paragraphs: paras,
          })
        }
        this.fileUploaded = true;
      }.bind(this);
      reader.readAsText(this.file);
    }

  },
  components: {
    AnnotationsPage,
    AnnotateArticle
  }
};
</script>

<style scoped>
.uploadBar {
  max-width: 300px;
}
.videop{
    align-content: center;
    margin-left: 20%;
    margin-right: 20%;
}

</style>


