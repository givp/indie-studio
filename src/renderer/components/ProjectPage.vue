<template>
  <div id="wrapper">
    <div id="top_nav">Home > <router-link :to="{ name: 'landing-page', params: {}}">Projects</router-link> > {{ project.project_name }}</div>

    <div id="tag_menu" ref="tag_menu">
        <button @click="doTag('a')">Props</button>
        <br>
        <button @click="doTag('b')">Stunts</button>
        <br>
        <button @click="doTag('c')">Sound FX</button>
        <br>
    </div>

    <main>
      <div v-if="project.project_name && !project.file">
        <div>Import your script (Final Draft .fdx file)</div>
        <div><button @click="openFile">Open</button></div>
      </div>
      <div class="scriptContent" v-if="content">

        <!-- Main script -->
        <div class="scriptInner">
          <div v-for="(item, index) in content">
            <div v-bind:class="item.$ && item.$.Type.replace(/ +/g, '')">{{ item.Text && item.Text[0] }}</div>
          </div>
        </div>
        <!-- End main script -->

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
  import tags from '../data/tags.js'

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
        tags: tags
      }
    },
    mounted: function () {
      var vm = this

      rangyHighlight.init();

      vm.highlighter = rangyHighlight.createHighlighter();

      vm.highlighter.addClassApplier(rangyHighlight.createClassApplier("a", {
          ignoreWhiteSpace: true,
          tagNames: ["span", "a"],
          elementProperties: {
              href: "#",
              onclick: function() {
                  var highlight = vm.highlighter.getHighlightForElement(this);
                  if (window.confirm("Delete this tag?")) {
                      vm.deleteTag(highlight)
                  }
                  return false;
              }
          }
      }));

      vm.highlighter.addClassApplier(rangyHighlight.createClassApplier("b", {
          ignoreWhiteSpace: true,
          tagNames: ["span", "a"]
      }));

      vm.highlighter.addClassApplier(rangyHighlight.createClassApplier("c", {
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
            vm.$db.update({_id: vm.id}, { $set: { file: true } }, {}, function(err, numReplaced){
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
              vm.highlighter.deserialize(vm.project.tags)
            }, 500)
          }

        })
      },
      updateFile() {
        var vm = this

        fs.writeFile(vm.file_path, JSON.stringify(vm.content, null, 2), function() {
          console.log('Data saved')
        });
      },
      saveTags() {
        var vm = this
        // update db
        vm.$db.update({_id: vm.id}, { $set: { tags: vm.highlighter.serialize() } }, {}, function(err, numReplaced){
          //
        })
      },
      deleteTag(highlight) {
        var vm = this
        vm.highlighter.removeHighlights( [highlight] );
        vm.saveTags();
      },
      doTag(tag) {
        var vm = this
        const selection = window.getSelection()
        vm.highlighter.highlightSelection(tag)
        vm.saveTags()
        selection.empty()
        vm.closeTagsMenu()
      },
      closeTagsMenu() {
        var tag_menu = document.getElementById("tag_menu")
        tag_menu.style.display = 'none'
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
        const oRect = selectionRange.getBoundingClientRect();

        if (startOffset == endOffset) {
          return
        }

        if (startNode.className != "Action" && startNode.className != "Dialogue" && startNode.className != "Parenthetical" && startNode.className != "Transition") {
          return
        }

        var tag_menu = document.getElementById("tag_menu")
        tag_menu.style.left = (oRect.left + (oRect.width/2)) - 100 + 'px';
        tag_menu.style.top = oRect.top + 20 + 'px'
        tag_menu.style.display = 'block'

        var rt = rangyTextRange.getSelection();
        rt.expand('word')
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

  .a {
    font-weight: bold;
    color: rgb(202, 68, 176);
    cursor: pointer;
  }

  .b {
    font-weight: bold;
    color: red;
    cursor: pointer;
  }

  .c {
    font-weight: bold;
    color: yellow;
    cursor: pointer;
  }

  #tag_menu {
    background-color: #ccc;
    width: 200px;
    height: 300px;
    position: absolute;
    display: none;
  }
</style>
