<template>
  <div class="AnnotationsPage container mt-3">
    <div>
      <b-navbar sticky toggleable="lg" class="pl-0">
        <b-navbar-brand>BAnno QA Annotator</b-navbar-brand>

        <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

        <b-collapse id="nav-collapse" is-nav>
          <!-- Right aligned nav items -->
          <b-navbar-nav class="ml-auto">
            <b-nav-item>
              <svg-progress-bar :value="data_number / json.data.length * 100" :options="options"></svg-progress-bar>
            </b-nav-item>
            <b-nav-form>
              <vue-bootstrap-typeahead
                size="sm"
                v-model="query"
                :data="autocomplete"
                placeholder="অনুচ্ছেদ খুঁজুন..."
                @hit="data_number = autocomplete.indexOf($event) + 1; context_number = 1"
              />
            </b-nav-form>
            <b-nav-item right>
              <b-button
                :size="'sm'"
                :variant="'primary'"
                v-on:click="delete_empty_document()"
                v-download-data="valid_json"
                v-download-data:type="'json'"
                v-download-data:filename="'cdqa-v1.1.json'"
              >ডাউনলোড</b-button>
            </b-nav-item>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </div>
    <br>
    <div v-if="data_number - 1 < json.data.length">
      <h2>{{ json.data[data_number - 1].title }}</h2>
      <span
        class="text-muted"
      >অনুচ্ছেদ {{ context_number }}/{{ json.data[data_number - 1].paragraphs.length }} | নিবন্ধ {{ data_number }}/{{ json.data.length }}</span>
      <br>
      <br>
      <p ref="paragraph" v-selection.fix="{getSelection:getSelection}">{{ paragraph_context }}</p>
      <br>

      <b-form-input v-model="question" type="text" placeholder="আপনার প্রশ্নটি টাইপ করুন..."></b-form-input>
      <br>

      <b-form-input v-model="answer" type="text" placeholder="আপনার উত্তরটি অনুচ্ছদটিতে নির্বাচন করুন..."></b-form-input>
      <br>

      <b-button :size="''" :variant="'secondary'" v-on:click="addAnnotation()">তথ্য যোগ করুন</b-button>
      <br>

      <b-table striped hover :items="items" :fields="fields">
        <template slot="সম্পাদনা করুন" slot-scope="row">
          <b-button :size="''" :variant="'danger'" @click.stop="deleteAnnotation(row.index)">মুছে ফেলুন</b-button>
        </template>
      </b-table>
      <br>

      <div v-if="data_number > 1 && context_number == 1">
        <b-button
          :size="''"
          :variant="'outline-secondary'"
          v-on:click="data_number -= 1, context_number = json.data[data_number - 1].paragraphs.length"
        >পূর্ববর্তী</b-button> or
        <b-button :size="''" :variant="'outline-primary'" v-on:click="context_number += 1">পরবর্তী</b-button>
      </div>
      <div v-else-if="context_number < json.data[data_number - 1].paragraphs.length">
        <b-button
          :size="''"
          :variant="'outline-secondary'"
          v-on:click="context_number -= 1"
        >পূর্ববর্তী</b-button> অথবা
        <b-button :size="''" :variant="'outline-primary'" v-on:click="context_number += 1">পরবর্তী</b-button>
      </div>
      <div v-else>
        <b-button
          :size="''"
          :variant="'outline-secondary'"
          v-on:click="context_number -= 1"
        >পূর্ববর্তী</b-button> অথবা
        <b-button
          :size="''"
          :variant="'outline-primary'"
          v-on:click="data_number += 1, context_number = 1"
        >পরবর্তী</b-button>
      </div>
      <br>
      <br>
    </div>
    <div v-else>
      টীকা করা ডেটা ডাউনলোড করুন।
      <img
        src="../assets/ablobmaracas.gif"
        height="30"
        width="30"
      >
      <br>
      <br>
      <b-button
        :size="''"
        :variant="'primary'"
        v-on:click="delete_empty_document()"
        v-download-data="valid_json"
        v-download-data:type="'json'"
        v-download-data:filename="'cdqa-v1.1.json'"
      >Download</b-button>
    </div>
  </div>
</template>

<script>

const uuidv4 = require('uuid/v4');

export default {
  name: "AnnotationsPage",
  props: ["json"],
  data: function() {
    return {
      data_number: 1,
      context_number: 1,
      question: "",
      answer: "",
      fields: ["প্রশ্ন", "উত্তর", "সম্পাদনা করুন"],
      query: ""
    };
  },
  methods: {
    addAnnotation: function() {
      var paragraph_container = this.json.data[this.data_number - 1].paragraphs[
        this.context_number - 1
      ];
      var qa = {
        question: this.question,
        id: uuidv4(),
        answers: [{ answer_start: this.answer_start, text: this.answer }]
      };
      paragraph_container.qas.push(qa);
      this.question = "";
      this.answer = "";
    },
    deleteAnnotation: function(row_index) {
      var paragraph_container = this.json.data[this.data_number - 1].paragraphs[
        this.context_number - 1
      ];
      paragraph_container.qas.splice(row_index, 1);
    },
    getSelection: function(fixStr) {
      this.answer = fixStr;
    },
    delete_paragraph: function() {
      var paragraph_container = this.json.data[this.data_number - 1].paragraphs;
      paragraph_container.splice([this.context_number - 1], 1);
    },
    delete_empty_document: function() {
      for (var i in this.json.data) {
        if (this.json.data[i].paragraphs.length == 0) {
          this.json.data.splice(i, 1);
        }
      }
    }
  },
  computed: {
    valid_json: function() {
      var json = JSON.stringify(this.json).replace(/[\u007F-\uFFFF]/g, function(
        chr
      ) {
        return "\\u" + ("0000" + chr.charCodeAt(0).toString(16)).substr(-4);
      });
      return json;
    },
    autocomplete: function() {
      var idx = [];
      for (var i = 0; i < this.json.data.length; i++) {
        idx.push(this.json.data[i].title);
      }
      return idx;
    },
    items: function() {
      var paragraph_container = this.json.data[this.data_number - 1].paragraphs[
        this.context_number - 1
      ];
      var items = [];
      for (var i = 0; i < paragraph_container.qas.length; i++) {
        var item = {
          প্রশ্ন: paragraph_container.qas[i].question,
          উত্তর: paragraph_container.qas[i].answers[0].text
        };
        items.push(item);
      }
      return items;
    },
    paragraph_context: function() {
      return this.json.data[this.data_number - 1].paragraphs[
        this.context_number - 1
      ].context;
    },
    answer_start: function() {
      return this.paragraph_context.indexOf(this.answer);
    },
    options() {
      return {
        radius: 15,
        circleLineCap: "round",
        varyStrokeArray: [1, 2],
        pathColors: ["lightgrey", "rgb(40, 167, 69)"],
        text: function(value) {
          return (
            this.htmlifyNumber(value) + '<span style="font-size: 1em;">%</span>'
          );
        },
        textColor: "black"
      };
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
