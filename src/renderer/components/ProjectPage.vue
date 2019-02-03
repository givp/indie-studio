<template>
  <div id="wrapper">
    <div id="top_nav">Tagger > <router-link :to="{ name: 'landing-page', params: {}}">Projects</router-link> > {{ project.project_name }}</div>

    <main>
      <div v-if="project.project_name && !project.file">
        <div>Import your script (Final Draft .fdx file)</div>
        <div><button @click="openFile">Open</button></div>
      </div>
      <div class="scriptContent" v-if="content">
        <input type="radio" id="highlight" value="highlight" v-model="picked">
        <label for="one">highlight</label>
        <br>
        <input type="radio" id="highlight2" value="highlight2" v-model="picked">
        <label for="two">highlight2</label>
        <br>
        <span>Picked: {{ picked }}</span>
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
  const uuidv4 = require('uuid/v4');
  import rangy from 'rangy';
  import rangyHighlight from 'rangy/lib/rangy-highlighter';
  import rangyClassApplier from 'rangy/lib/rangy-classapplier';
  import rangyTextRange from 'rangy/lib/rangy-textrange';
  import rangySerializer from 'rangy/lib/rangy-serializer';

  export default {
    name: 'project-page',
    data () {
      return {
        id: this.$route.params.id,
        file_path: path.join(remote.app.getPath('userData'), this.$route.params.id + '.json'),
        project: {},
        debugText: "",
        highlighter: null,
        content: null,
        picked: "highlight"
      }
    },
    mounted: function () {
      var vm = this

      rangyHighlight.init();
      rangy.init();

      vm.highlighter = rangyHighlight.createHighlighter();

      vm.highlighter.addClassApplier(rangyHighlight.createClassApplier("highlight", {
          ignoreWhiteSpace: true,
          tagNames: ["span", "a"]
      }));

      vm.highlighter.addClassApplier(rangyHighlight.createClassApplier("highlight2", {
          ignoreWhiteSpace: true,
          tagNames: ["span", "a"]
      }));

      vm.$db.find({_id: vm.id }).exec(function (err, docs) {
        vm.project = docs[0]
        if (vm.project.file) {
          vm.displayFile()
        }
      });

      window.addEventListener('mouseup', this.onMouseup)
    },
    beforeDestroy: function() {
      window.removeEventListener('mouseup', this.onMouseup)
    },
    methods: {
      generateId() {
        return uuidv4()
      },
      openFile() {
        var vm = this
        dialog.showOpenDialog({ properties: ['openFile'], filters: [
          { name: 'Final Draft', extensions: ['fdx'] }
        ]
        }, this.initFile)
      },
      initFile(paths) {
        var vm = this

        if (!paths) {
          return
        }

        var originalPath = paths[0]

        fs.readFile(originalPath, 'utf-8', (err, data) => {
          if(err){
              alert("An error ocurred reading the file :" + err.message);
              return;
          }

          parseString(data, function (err, result) {
            var xmlContent = result.FinalDraft.Content[0].Paragraph

            fs.writeFile(vm.file_path, JSON.stringify(xmlContent, null, 2));

            // update db
            vm.$db.update({_id: vm.id}, { $set: { file: true } }, function(){
              vm.project.file = true
              vm.displayFile()
            })

          })

        })

      },
      displayFile() {
        var vm = this

        fs.readFile(vm.file_path, 'utf-8', (err, data) => {
          if(err){
              alert("An error ocurred reading the file :" + err.message);
              return;
          }

          vm.content = JSON.parse(data)

          if (vm.project.tags && vm.project.tags.length) {
            setTimeout(function(){
              console.log("Loading")
              vm.highlighter.deserialize(vm.project.tags)
            }, 100)
          }

        })
      },
      updateFile() {
        var vm = this

        fs.writeFile(vm.file_path, JSON.stringify(vm.content, null, 2), function() {
          console.log('Data saved')
        });
      },
      onMouseup () {
        var vm = this

        const selection = window.getSelection()
        if (!selection || selection.rangeCount < 1) {
          return
        }

        const selectionRange = selection.getRangeAt(0)
        const startNode = selectionRange.startContainer.parentNode
        const endNode = selectionRange.endContainer.parentNode
        const startOffset = selectionRange.startOffset
        const endOffset = selectionRange.endOffset
        const tagId = vm.generateId();

        if (startOffset == endOffset) {
          return
        }

        if (startNode.className != "Action" && startNode.className != "Dialogue" && startNode.className != "Parenthetical" && startNode.className != "Transition") {
          return
        }

        vm.highlighter.highlightSelection(vm.picked);

        // update db
        vm.$db.update({_id: vm.id}, { $set: { tags: vm.highlighter.serialize() } }, {}, function(){
          console.log("saved")
        })

        selection.empty()
      }
    }
  }
</script>

<style lang="scss">

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

  .highlight2 {
    font-weight: bold;
    color: red;
    cursor: pointer;
  }
</style>
