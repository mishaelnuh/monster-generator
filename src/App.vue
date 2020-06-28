<template>
  <v-app>
    <v-app-bar app dense elevate-on-scroll dark>
      <i class="fas fa-pastafarianism fa-lg mr-4"></i>
      <v-toolbar-title>Monster Generator</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn @click="exportCards">
        <i class="fas fa-file-export fa-lg mr-2"></i> Export All
      </v-btn>
    </v-app-bar>
    <v-content>
      <v-container style="padding-bottom: 30px;">
        <v-row>
          <v-col xs="6" sm="6" md="4" lg="3" xl="2">
            <v-card outlined style="height: 100%;">
              <v-responsive>
                <v-card-title>
                  Generate a Monster
                </v-card-title>
                <v-card-text class="caption">
                  Utilises the following APIs to generate monster cards:
                  <ul>
                    <li><a href="https://monsternames-api.com">monsternames-api.com</a></li>
                    <li><a href="https://robohash.org/">robohash.org</a></li>
                    <li><a href="https://thesimpsonsquoteapi.glitch.me/">thesimpsonsquoteapi.glitch.me</a></li>
                  </ul>
                  <v-divider class="my-4"></v-divider>
                  <div class="overline">Select monster type</div>
                  <v-chip-group column mandatory active-class="primary--text" v-model="selectedMonsterTab">
                    <v-chip v-for="type in monsterTypes" :key="type.name" small>{{type.name}}</v-chip>
                  </v-chip-group>
                </v-card-text>
                <v-card-actions>
                  <v-btn large text @click="getMonster">
                    Generate
                  </v-btn>
                  <v-btn large text @click="removeAllCards">
                    Clear all
                  </v-btn>
                </v-card-actions>
              </v-responsive>
            </v-card>
          </v-col>
          <v-col xs="6" sm="6" md="4" lg="3" xl="2" v-for="(monster, index) in generatedMonsters.slice().reverse()" :key="index" ref="generatedMonsters">
            <v-card :dark="monster.darkCard" :color="monster.backgroundColor">
              <v-responsive :aspect-ratio="5/7">
                <v-card-title>
                  <div style="width: 100%; text-overflow: ellipsis; white-space: nowrap; overflow: hidden;">
                    {{monster.name.fullName}}
                  </div>
                </v-card-title>
                <v-card-subtitle class="overline">{{monster.type}}</v-card-subtitle>
                <v-container class="pt-0" style="height: 100%;">
                  <v-img :src="monster.roboHashPath" style="margin-top: -40px;" contain></v-img>
                  <v-card-text class="mx-auto mt-3 pt-1 pb-0 black--text pa-2 caption" style="background: #fafafa; height: 100%; border-radius: 6px 6px 0 0; line-height: 1.1;">
                    <i>
                      {{monster.quote}}
                    </i>
                  </v-card-text>
                </v-container>
                <div v-show="!hideCardOrnaments">
                  <v-hover v-slot:default="{ hover }">
                    <v-btn x-small absolute top right depressed color="error" class="mr-n5" @click="removeCard(index)">
                      <v-icon x-small class="mr-1">mdi-delete</v-icon> {{ hover ? 'delete' : ''}}
                    </v-btn>
                  </v-hover>
                  <v-hover v-slot:default="{ hover }">
                    <v-btn x-small absolute top right depressed color="success" class="mt-6 mr-n5" @click="exportCard(index)">
                      <v-icon x-small class="mr-1">mdi-download</v-icon> {{ hover ? 'export' : ''}}
                    </v-btn>
                  </v-hover>
                </div>
              </v-responsive>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
      <v-footer dark padless absolute style="height: 30px;" class="px-3">
        <div class="overline">
          Made by <a href="https://www.mnuh.org/" style="color: white;">Mishael Nuh</a> 
        </div>
        <v-spacer></v-spacer>
        <v-btn icon small href="https://github.com/mishaelnuh/monster-generator"><i class="fab fa-github"></i></v-btn>
      </v-footer>
      <v-overlay :value="overlay" opacity="0.9">
        <v-col text-align="center">
          <div class="overline">Exporting...</div>
          <br/>
          <v-progress-circular indeterminate :size="70"></v-progress-circular>
        </v-col>
      </v-overlay>
    </v-content>
  </v-app>
</template>

<script>
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'

export default {
  name: 'App',

  components: {
  },

  data: () => ({
    monsterTypes: [
      { name: 'Goatmen', endpoint: '/api/v1.0/goatmen' },
      { name: 'Goblin', endpoint: '/api/v1.0/goblin' },
      { name: 'Ogre', endpoint: '/api/v1.0/ogre' },
      { name: 'Orc', endpoint: '/api/v1.0/orc' },
      { name: 'Skeleton', endpoint: '/api/v1.0/skeleton' },
      { name: 'Troll', endpoint: '/api/v1.0/troll' },
    ],
    currentMonsterType: null,
    selectedMonsterTab: 0,
    generatedMonsters: [],
    hideCardOrnaments: false,
    overlay: false,
  }),
  watch: {
    generatedMonsters(updatedMonsters) {
      localStorage.setItem('generatedMonsters', JSON.stringify(updatedMonsters))
    },
    selectedMonsterTab(updatedTab) {
      localStorage.setItem('selectedMonsterTab', JSON.stringify(updatedTab))
    }
  },
  methods: {
    removeCard(index) {
      this.generatedMonsters.splice(this.generatedMonsters.length - index - 1, 1)
    },
    removeAllCards() {
      this.generatedMonsters.splice(0, this.generatedMonsters.length)
    },
    getMonster() {
      const randomColor = this.getRandomColor()
      const fetchName = fetch('https://monsternames-api.com' + this.monsterTypes[this.selectedMonsterTab].endpoint)
        .then(res => {
          return res.json()
        })
      const fetchQuote = fetch('https://thesimpsonsquoteapi.glitch.me/quotes')
        .then(res => {
          return res.json()
        })
      Promise.all([fetchName, fetchQuote])
        .then(vals => {
          return {
            name: vals[0],
            type: this.monsterTypes[this.selectedMonsterTab].name,
            quote: vals[1][0].quote,
            backgroundColor: '#' + randomColor,
            darkCard: this.isDarkCard(randomColor),
            roboHashPath: 'https://robohash.org/' + encodeURIComponent(vals[0].fullName) + '?set=set2'
          }
        })
        .then(newCard => {
          this.toDataURL(newCard.roboHashPath)
            .then(res => {
              newCard.roboHashPath = res
              this.generatedMonsters.push(newCard)
            })
        })
      },
    getRandomColor() {
      const letters = '0123456789ABCDEF'
      var color = ''
      for (var i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)]
      }
      return color
    },
    isDarkCard(hex) {
      const rLinear = Math.pow(parseInt('0x' + hex.substring(0, 2)) / 255, 2.2)
      const gLinear = Math.pow(parseInt('0x' + hex.substring(2, 4)) / 255, 2.2)
      const bLinear = Math.pow(parseInt('0x' + hex.substring(4, 6)) / 255, 2.2)
      const y = 0.2126*rLinear + 0.7152*gLinear + 0.0722*bLinear
      
      return y < 0.5
    },
    exportCard(index) {
      this.overlay = true

      var doc = new jsPDF('p', 'in', [180, 252])
      var docWidth = doc.internal.pageSize.getWidth()
      var docHeight = doc.internal.pageSize.getHeight()
      this.hideCardOrnaments = true

      var canvasElement = document.createElement('canvas')
      html2canvas(this.$refs.generatedMonsters[index], {
        canvas: canvasElement,
        scale: 3,
      })
        .then(canvas => {
          this.hideCardOrnaments = false
          const image = canvas.toDataURL('image/jpeg', 1)
          doc.addImage(image, 'JPEG', 0, 0, docWidth, docHeight)
          doc.save(this.generatedMonsters[this.generatedMonsters.length - 1 - index].name.fullName + '.pdf')
          this.overlay = false
        })
    },
    exportCards() {
      if (this.generatedMonsters.length == 0) {
        return
      }

      this.overlay = true

      var doc = new jsPDF('p', 'in', [180, 252])
      var docWidth = doc.internal.pageSize.getWidth()
      var docHeight = doc.internal.pageSize.getHeight()

      var promiseArray = []
      this.hideCardOrnaments = true
      for (var i = 0; i < this.generatedMonsters.length; i++) {
        var canvasElement = document.createElement('canvas')
        promiseArray.push(
          html2canvas(this.$refs.generatedMonsters[i], {
            canvas: canvasElement,
            scale: 3,
          })
            .then(canvas => {
              const image = canvas.toDataURL('image/jpeg', 1)
              doc.addImage(image, 'JPEG', 0, 0, docWidth, docHeight)
              doc.addPage()
            })
        )
      }

      Promise.all(promiseArray.reverse()).then(() => {
        this.hideCardOrnaments = false
        doc.deletePage(this.generatedMonsters.length + 1)
        doc.save('monster-cards.pdf')
        this.overlay = false
      })
    },
    toDataURL(src, outputFormat) {
      return new Promise(resolve => {
        var img = new Image()
        img.crossOrigin = 'Anonymous'
        img.addEventListener('load', () => {
          var canvas = document.createElement('CANVAS')
          var ctx = canvas.getContext('2d')
          canvas.height = img.naturalHeight
          canvas.width = img.naturalWidth
          ctx.drawImage(img, 0, 0)
          resolve(canvas.toDataURL(outputFormat))
        })
        img.src = src
      })
    }
  },
  mounted() {
    if (localStorage.generatedMonsters) {
      this.generatedMonsters = JSON.parse(localStorage.getItem('generatedMonsters'))
    }
    if (localStorage.selectedMonsterTab) {
      this.selectedMonsterTab = JSON.parse(localStorage.getItem('selectedMonsterTab'))
    }
  }
};
</script>
