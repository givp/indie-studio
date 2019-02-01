<template>
  <div id="wrapper">
    <main>
      <div>Import your script (Final Draft .fdx file)</div>
      <div><button @click="openFile">Open</button></div>
      <router-link :to="{ name: 'landing-page', params: {}}">Projects</router-link>

      <div class="scriptContent" v-if="content">
        <div v-for="(item, index) in content">
          <div v-bind:class="item.$ && item.$.Type.replace(/ +/g, '')">{{ item.Text && item.Text[0] }}</div>
        </div>
      </div>

    </main>
  </div>
</template>

<script>
  const { dialog } = require('electron').remote;
  var fs = require('fs');
  var parseString = require('xml2js').parseString;

  export default {
    name: 'project-page',
    data () {
      return {
        debugText: "",
        content: null
      }
    },
    mounted: function () {
      //this.$db.insert({test: 234234})
      this.parseFile(["/Users/giv/Desktop/dig.fdx"])
      window.addEventListener('mouseup', this.onMouseup)
    },
    beforeDestroy: function() {
      window.removeEventListener('mouseup', this.onMouseup)
    },
    methods: {
      openFile() {
        var vm = this
        dialog.showOpenDialog({ properties: ['openFile'], filters: [
          { name: 'Final Draft', extensions: ['fdx'] }
        ]
        }, this.parseFile)
      },
      parseFile(paths) {
        var vm = this

        if (!paths) {
          return
        }

        var xmlPath = paths[0]
        console.log(xmlPath)
        fs.readFile(xmlPath, 'utf-8', (err, data) => {
          if(err){
              alert("An error ocurred reading the file :" + err.message);
              return;
          }

          parseString(data, function (err, result) {
            vm.content = result.FinalDraft.Content[0].Paragraph
          });
        });
      },
      onMouseup () {
        const selection = window.getSelection()
        if (!selection || selection.rangeCount < 1) {
          return
        }

        const selectionRange = selection.getRangeAt(0)
        const startNode = selectionRange.startContainer.parentNode
        const endNode = selectionRange.endContainer.parentNode

        if (selectionRange.startOffset == selectionRange.endOffset) {
          return
        }

        if (startNode.className != "Action" && startNode.className != "Dialogue" && startNode.className != "Parenthetical") {
          return
        }

        var selectedText = selectionRange.extractContents();
        var span= document.createElement("span");
        span.className = "highlight";
        span.appendChild(selectedText);
        selectionRange.insertNode(span);

        //console.log(startNode.className)
        //console.log(selection.toString())
        selection.empty()
      }
    }
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body {
    font-family: 'Source Sans Pro', sans-serif;
    background-color: #e6e6e6;
    font-color: #333;
    padding: 50px;
  }

  #wrapper {

  }
  main > div { flex-basis: 50%; }

  .scriptContent {
    font-family: "CourierPrime", monospace;
    font-size: 15px;
    margin-top: 20px;
    font-weight: 500;
    background: #fff;
    width: 750px;
    padding: 50px;
  }

  .SceneHeading {
    margin-top: 40px;
    font-weight: bold;
  }

  .Action {
    margin-top: 25px;
  }

  .Character {
    text-align: center;
    margin-top: 20px;
  }

  .Dialogue {
    text-align: center;
  }

  .Parenthetical {
    text-align: center;
    font-style: italic;
  }

  .highlight {
    font-weight: bold;
    color: rgb(202, 68, 176);
  }
</style>
