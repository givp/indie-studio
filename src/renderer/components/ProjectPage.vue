<template>
  <div id="wrapper">
    <div id="top_nav">Tagger > <router-link :to="{ name: 'landing-page', params: {}}">Projects</router-link> > {{ project.project_name }}</div>

    <main>
      <div v-if="project.project_name && !project.file">
        <div>Import your script (Final Draft .fdx file)</div>
        <div><button @click="openFile">Open</button></div>
      </div>
      <div class="scriptContent" v-if="content">
        <div class="scriptInner">
          <div v-for="(item, index) in content">
            <div v-bind:class="item.$ && item.$.Type.replace(/ +/g, '')">{{ item.Text && item.Text[0] }}</div>
          </div>
        </div>
      </div>

      <div class="scriptInfo" v-if="content">
        <ul v-for="(item, index) in content">
          <li v-if="item.$.Type && item.$.Type == 'Scene Heading'">
            <div class="rowScene">
              <div>{{ item.$.Number }} - {{ item.Text[0] }} - {{ item.SceneProperties[0].$.Length }}</div>
            </div>
          </li>
        </ul>
      </div>

    </main>
  </div>
</template>

<script>
  // { "$": { "Type": "Scene Heading", "Number": "1" }, "SceneProperties": [ { "$": { "Length": "7/8", "Page": "1", "Title": "" } } ], "Text": [ "EXT. TOP OF HILL - DAY" ] }
  const { dialog } = require('electron').remote;
  var fs = require('fs');
  var parseString = require('xml2js').parseString;
  import path from 'path'
  import { remote } from 'electron'

  export default {
    name: 'project-page',
    data () {
      return {
        id: this.$route.params.id,
        project: {},
        debugText: "",
        content: null
      }
    },
    mounted: function () {
      var vm = this
      vm.$db.find({_id: vm.id }).exec(function (err, docs) {
        vm.project = docs[0]
        if (vm.project.file) {
          vm.displayFile()
        }
      });

      //this.parseFile(["/Users/giv/Desktop/dig.fdx"])
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
        }, this.saveFile)
      },
      saveFile(paths) {
        var vm = this

        if (!paths) {
          return
        }

        var originalPath = paths[0]
        var savePath = path.join(remote.app.getPath('userData'), vm.id + '.xml')

        fs.copyFile(originalPath, savePath, function(e){
          vm.$db.update({_id: vm.id}, { $set: { file: true } }, function(){
            vm.project.file = true
            vm.displayFile()
          })
        })

      },
      displayFile() {
        var vm = this

        var savePath = path.join(remote.app.getPath('userData'), vm.id + '.xml')
        fs.readFile(savePath, 'utf-8', (err, data) => {
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

  li {
    list-style-type: none;
    padding: 0;
    margin: 0;
  }

  .rowScene {
    height: 40px;
    border-top: 1px solid #50898A;
    line-height: 40px;
    font-size: 12px;
    color: #4C5359;
  }

  .scriptContent {
    font-family: "CourierPrime", monospace;
    font-size: 15px;
    font-weight: 500;
    background: #fff;
    max-width: 816px;
    min-width: 600px;
    padding: 50px;
    height: 100vh;
    overflow-y: scroll;
  }

  .scriptInner {
    margin-right: 86px;
    margin-left: 86px;
    padding-left: 0!important;
    padding-right: 0!important;
    max-width: 100%;
    color: #333;
  }

  .SceneHeading {
    margin-top: 40px;
    font-weight: bold;
  }

  .scriptInfo {
    margin-left: 20px;
  }

  .Action {
    margin-top: 25px;
  }

  .Character {
    text-align: center;
    margin-top: 20px;
  }

  .Transition {
    text-align: right;
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
    cursor: pointer;
  }
</style>
