<template>
  <v-app>
    <v-app-bar app dense elevate-on-scroll dark collapse-on-scroll>
      <i class="fas fa-pastafarianism fa-lg mr-4"></i>
      <v-toolbar-title>Monster Generator</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn icon href="https://github.com/mishaelnuh/monster-generator">
        <i class="fab fa-github fa-lg"></i>
      </v-btn>
    </v-app-bar>
    <v-content>
      <v-container fluid>
        <v-row>
          <v-col xs="12" sm="6" md="4" lg="3" xl="2">
            <v-card outlined style="height: 100%;">
              <v-responsive :aspect-ratio="5/7">
                <v-card-title>
                  Generate a Monster
                </v-card-title>
                <v-card-text>
                  Utilises the following APIs to generate monster cards:
                  <ul>
                    <li><a href="https://monsternames-api.com">monsternames-api.com</a></li>
                    <li><a href="https://thesimpsonsquoteapi.glitch.me/">thesimpsonsquoteapi.glitch.me</a></li>
                  </ul>
                  <v-divider class="my-4"></v-divider>
                  Select monster type
                  <v-chip-group column mandatory active-class="primary--text" v-model="selectedMonsterTab">
                    <v-chip v-for="type in monsterTypes" :key="type.name">{{type.name}}</v-chip>
                  </v-chip-group>
                </v-card-text>
                <v-card-actions>
                  <v-btn large text @click="getMonster">
                    Generate
                  </v-btn>
                  <v-btn large text @click="removeAll">
                    Clear all
                  </v-btn>
                </v-card-actions>
              </v-responsive>
            </v-card>
          </v-col>
          <v-col xs="12" sm="6" md="4" lg="3" xl="2" v-for="(monster, index) in generatedMonsters.slice().reverse()" :key="index">
            <v-card :dark="monster.darkCard" :color="monster.backgroundColor">
              <v-responsive :aspect-ratio="5/7">
                <v-row>
                  <v-col cols="11" class="py-0">
                    <v-card-title>{{monster.name.fullName}}</v-card-title>
                    <v-card-subtitle>{{monster.type}}</v-card-subtitle>
                  </v-col>
                  <v-col cols="1">
                    <v-btn x-small light depressed absolute right fab @click="removeCard(index)">
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>
                  </v-col>
                </v-row>
                <v-container style="height: 100%;">
                  <v-img :src="monster.roboHashPath" style="margin-top: -30px;" contain></v-img>
                  <v-card-text class="mx-auto mt-4 pb-0 black--text pa-2" style="background: #fafafa; height: 100%; border-radius: 6px 6px 0 0; text-align: justify; font-size: 60%; line-height: 130%;">
                    <i>
                      {{monster.quote}}
                    </i>
                  </v-card-text>
                </v-container>
              </v-responsive>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
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
    generatedMonsters: []
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
    removeAll() {
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
      Promise.all([fetchName, fetchQuote]).then(vals =>{
        this.generatedMonsters.push({
          name: vals[0],
          type: this.monsterTypes[this.selectedMonsterTab].name,
          quote: vals[1][0].quote,
          backgroundColor: '#' + randomColor,
          darkCard: this.isDarkCard(randomColor),
          roboHashPath: 'https://robohash.org/' + encodeURIComponent(vals[0].fullName) + '?set=set2'
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
