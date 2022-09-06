<template>
  <div class="hello">
    <h1 class="headtext">{{ msg }}</h1>
    <span id="inputshow"></span>
    <textarea
      placeholder="Enter the Text"
      class="form-control"
      id="maintextarea"
      v-model="spellval"
    ></textarea>

    <div class="Suggestionbox" v-show="wordCorrectCloud.length > 0">
      <h2 class="Suggestiontext">Suggestion</h2>

      <div>
        <button
          class="btn Suggestionbtn"
          @click="changecorrectword(word)"
          v-for="(word, i) in wordCorrectCloud"
          :key="i"
        >
          {{ word }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
// import func from 'vue-editor-bridge';
export default {
  data() {
    return {
      name: "HelloWorld",
      curmsg: "",
      indexarr: "",
      selectedval: "",
      spellval: "",
      wordCloud: [],
      wordCorrectCloud: [],
    };
  },

  props: {
    msg: String,
  },
  created() {},
  watch: {},
  mounted() {
    this.getSelection();
  },
  methods: {
    spellcheck() {
      if (this.spellval.length > 0) {
        this.wordCloud = this.spellval.split(" ");
      }
      //console.log(this.wordCloud);
      var self = this;
      fetch(
        "https://api.textgears.com/spelling?key=wejm8cfYGDAp4JT4&text=" +
          this.spellval +
          "&language=en-GB"
      )
        .then((response) => response.json())
        .then((data) => {
          console.log("testexample" + data);
          self.curmsg = document.getElementById("inputshow").innerHTML;
          //console.log(data.response.errors)

          let badword = self.curmsg;
          for (const w in data.response.errors) {
            let f1 = data.response.errors[w];
            if (f1.bad == badword) {
              self.indexarr = w;
              //  console.log(w)
            }
          }

          //let index = data.response.errors.indexOf(self.curmsg);
          if (
            data.response.errors &&
            data.response.errors[self.indexarr].length > 0
          ) {
            let content = data.response.errors[self.indexarr];

            // console.log(content)
            self.wordCorrectCloud = content.better;

            //let badword = self.curmsg
            //// console.log(badword, 'badword')
            //  console.log(content.bad, 'contentbad')

            //  for(const w in content) {
            //      let f1 = content[w];
            //      console.log(f1, 'f1')
            //      if(content.bad == badword) {
            //          self.wordCorrectCloud = content.better;
            //          console.log(self.wordCorrectCloud)
            //    }
            //  }

            // for(const w in self.wordCorrectCloud) {
            // self.wordCorrectCloud = data.response.errors[0].better;
            // self.wordbadCloud = data.response.errors[0].bad;
            // console.log(self.wordbadCloud);
            // }
            //           for(const w in self.wordCorrectCloud) {
            //             let f1 = self.wordCorrectCloud;
            //             if(f1.bad == self.curmsg) {
            //                 self.wordCorrectCloud = f1.better;
            //             }
            // }
          }
        });
    },
    getSelection: function () {
      var self = this;
      let input = document.querySelector("textarea");

      input.addEventListener(
        "click",
        function () {
          // We only need the character offset here
          // Assume all non-word character(s) is a pipe
          let test = this.value.replace(/\W/g, "|");
          let caret = this.selectionStart;
          // Get the last word part length before caret
          let indexBeforeCaret = test
            .substring(0, caret)
            .split("|")
            .pop().length;
          // Get the first word part length after caret
          let indexAfterCaret = test.substring(caret).split("|")[0].length;
          // let word = this.value.substring(caret - indexBeforeCaret, caret + indexAfterCaret);
          // console.log(word);
          // Expand current caret position to the whole word
          this.selectionStart = caret - indexBeforeCaret;
          this.selectionEnd = caret + indexAfterCaret;
          this.curmsg = document.getElementById("inputshow").innerHTML;
          // this.spellcheck(this.curmsg);

          // let word = "suggeston"
          // let wordCorrectCloud = []

          const [$show, $input] = document.querySelectorAll("span,textarea");
          const getWord = (s, pos) => {
            const n = s.substring(pos).match(/^[a-zA-Z0-9-_]+/);
            const p = s.substring(0, pos).match(/[a-zA-Z0-9-_]+$/);

            // if you really only want the word if you click at start or between
            // but not at end instead use if (!n) return
            if (!p && !n) return "";
            return (p || "") + (n || "");
          };

          const showWord = (e) =>
            ($show.innerText = getWord(
              e.target.value,
              e.target.selectionStart
            ));
          $input.addEventListener("click", showWord);
          $input.addEventListener("input", showWord);

          self.spellcheck();
          // let wordF = getWord(e.target.value, e.target.selectionStart);
        },
        false
      );
    },

    changecorrectword(text) {
      function getInputSelection(el) {
        var start = 0,
          end = 0,
          normalizedValue,
          range,
          textInputRange,
          len,
          endRange;

        if (
          typeof el.selectionStart == "number" &&
          typeof el.selectionEnd == "number"
        ) {
          start = el.selectionStart;
          end = el.selectionEnd;
        } else {
          range = document.selection.createRange();

          if (range && range.parentElement() == el) {
            len = el.value.length;
            normalizedValue = el.value.replace(/\r\n/g, "\n");

            // Create a working TextRange that lives only in the input
            textInputRange = el.createTextRange();
            textInputRange.moveToBookmark(range.getBookmark());

            // Check if the start and end of the selection are at the very end
            // of the input, since moveStart/moveEnd doesn't return what we want
            // in those cases
            endRange = el.createTextRange();
            endRange.collapse(false);

            if (textInputRange.compareEndPoints("StartToEnd", endRange) > -1) {
              start = end = len;
            } else {
              start = -textInputRange.moveStart("character", -len);
              start += normalizedValue.slice(0, start).split("\n").length - 1;

              if (textInputRange.compareEndPoints("EndToEnd", endRange) > -1) {
                end = len;
              } else {
                end = -textInputRange.moveEnd("character", -len);
                end += normalizedValue.slice(0, end).split("\n").length - 1;
              }
            }
          }
        }

        return {
          start: start,
          end: end,
        };
      }

      function replaceSelectedText(el, text) {
        var sel = getInputSelection(el),
          val = el.value;
        el.value = val.slice(0, sel.start) + text + val.slice(sel.end);
      }

      var el = document.getElementById("maintextarea");
      replaceSelectedText(el, text);
      this.spellval = el.value;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.Suggestionbox {
  text-align: left;
}
#inputshow {
  visibility: hidden;
}
.Suggestiontext {
  text-align: left;
  font-size: 14px;
  color: #ffffff;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
.btn {
  display: inline-block;
  font-weight: 400;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  border: 1px solid transparent;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: 0.25rem;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out,
    border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.Suggestionbtn {
  background-color: #ffffff;
  font-size: 12px;
  font-weight: 600;
  margin: 0px 4px;
  padding: 6px 20px;
  letter-spacing: 0.3px;
  color: #15698b;
  cursor: pointer;
}
.Suggestionbtn:hover {
  background-color: #15698b;
  color: #ffffff;
}
#maintextarea {
  height: 300px;
  width: 100%;
}
.hello {
  width: 80%;
  margin: auto;
}
.headtext {
  color: #ffffff;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.form-control {
  display: block;
  width: 100%;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 0.25rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  width: 80%;
  margin: auto;
}

@import url("https://fonts.googleapis.com/css?family=Lato");
* {
  box-sizing: border-box;
  -webkit-animation: fadeIn 0.5s;
  animation: fadeIn 0.5s;
}
body {
  background-color: #52b3d9;
  color: #fafafa;
  text-align: center;
  font-size: 16px;
  font-family: "Lato", sans-serif;
}
p {
  max-width: 300px;
  margin: 0 auto 30px;
}
h1 {
  font-size: 2.5rem;
}
textarea {
  height: 150px;
}
.demo-input {
  margin: 10px auto;
}
.use-cases {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
}
.use-cases .use-case {
  margin: 0 15px;
}
.custom-suggestions {
  cursor: pointer;
  list-style: none;
  margin: 0;
  padding: 0;
}
.custom-suggestions--active {
  background-color: #19b5fe;
}
.custom-suggestions li {
  text-align: left;
}
.custom-suggestions li:hover {
  background-color: #19b5fe;
}
.input {
  border: 0;
  border-radius: 4px;
  color: #111;
  font-size: 16px;
  line-height: 16px;
  outline: 0;
  padding: 10px 12px;
  position: relative;
  text-align: left;
  width: 230px;
}
.input::-moz-placeholder {
  color: #cfcfcf;
  text-align: center;
}
.input:-ms-input-placeholder {
  color: #cfcfcf;
  text-align: center;
}
.input::placeholder {
  color: #cfcfcf;
  text-align: center;
}
.input__marker {
  background-color: #111;
  border-radius: 4px;
  box-shadow: 0 2px 5px #585858;
  display: block;
  font-size: 12px;
  padding: 4px 6px;
  position: absolute;
  transform: translate(5%, -70%);
  transition: top 0.1s ease 0s, left 0.1s ease 0s;
  white-space: nowrap;
  width: auto;
  z-index: 9999;
}
.input__marker:after {
  background-color: #111;
  content: "";
  height: 10px;
  position: absolute;
  width: 15px;
  z-index: -1;
}
.input__marker--position:after {
  bottom: 0;
  left: 0;
  transform: translate(-10%, 10%) rotate(-15deg) skewX(-40deg);
}
.input__marker--selection {
  transform: translate(-50%, -100%);
}
.input__marker--selection:after {
  bottom: 0;
  left: 50%;
  transform: translate(-50%, 0) rotate(45deg);
}
.input__marker--custom {
  transform: translate(16px, 16px);
}
.input__marker--custom:after {
  display: none;
}
@-webkit-keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
