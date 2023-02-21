<template>
  <div v-if="startAnnotate">
    <AnnotateCustomData v-bind:json="json"/>
  </div>
  <div v-else>
    <input id="articletitle" placeholder="নিবন্ধের শিরোনাম"/><br><br>
    <input id="articleurl" placeholder="নিবন্ধের URL"/><br><br>
    <span>নীচের উইকিপিডিয়া নিবন্ধের বিষয়বস্তু অনুলিপি করুন (একটি নতুন লাইন দ্বারা পৃথক প্রতিটি অনুচ্ছেদ লিখুন):</span>
    <br>
    <textarea id="txtarea" placeholder="নিবন্ধের বিষয়বস্তু"></textarea>
    <div>
      <b-button :size="''" :variant="'warning'" v-on:click="annotateData()">টীকা করুন</b-button>&nbsp;&nbsp;&nbsp;
    </div>

  </div>
</template>

<script>
import AnnotateCustomData from "./AnnotateCustomData";

export default {
  name: "AnnotateArticle",
  data: function (){
    return {
      startAnnotate: false,
      json: {
        data: [],
      },
      file: null,
    }
  },
  methods: {
    annotateData: function (){

      let inputarr = document.getElementById("txtarea").value.split("\n");
      let arttitle = document.getElementById("articletitle").value;
      let arturl = document.getElementById("articleurl").value;

      if (arttitle===""){
        arttitle="n/a";
      }
      if (arturl===""){
        arturl="n/a";
      }


      let paras = [];
      // eslint-disable-next-line no-unused-vars
      let prev_text = "";
      let current_text = "";
      // /* eslint-disable no-console */
      // console.log(this.json.data.length);
      for (var i = 0; i < inputarr.length; i++) {
        if (inputarr[i].trim()==="") {
          prev_text = current_text;

          if(current_text!==""){
              paras.push({
              context: current_text,
              qas: [],
            });
          }

          current_text = "";
        }else{
          if (current_text!==""){
            current_text = current_text.trim() +' '+inputarr[i].trim();
          }
          else{
            current_text = inputarr[i].trim();
          }

        }
      }


      if (current_text!== prev_text && current_text!==""){
        paras.push({
            context: current_text,
            qas: [],
          });
      }

      this.json.data.push({
        title: arttitle,
        url: arturl,
        paragraphs: paras,
      });

      document.getElementById("txtarea").value = "";
      document.getElementById("articleurl").value = "";
      document.getElementById("articletitle").value="";
      if(this.json.data.length===0){
        alert("No paragraphs found !!!");
      }else{
        /* eslint-disable no-console */
        console.log(this.json.data.length);
        this.startAnnotate = true;
      }

      return this.json;
    },
  },
  components: {
    AnnotateCustomData
  }
}
</script>

<style scoped>
  body{
    color: rgb(55, 53, 47);
  fill: currentcolor;
  line-height: 1.5;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, "Apple Color Emoji", Arial, sans-serif, "Segoe UI Emoji", "Segoe UI Symbol";
  -webkit-font-smoothing: auto;
  }
  p {
    white-space: pre-wrap;
  }
  #articletitle{
    min-width: 85%;
  }

  #articleurl{
    min-width: 60%;
  }

  #txtarea{
    padding:10px;
    margin:10px 0;
    width: 85%;
    min-height: 500px;
    max-height: 80%;
    transition: 180ms box-shadow ease-in-out;
  }
</style>
