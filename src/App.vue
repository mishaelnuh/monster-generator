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
              <v-card-title>
                Generate a Monster
              </v-card-title>
              <v-card-text>
                Utilises the following APIs to generate monster cards:
                <ul>
                  <a href="https://monsternames-api.com"><li>monsternames-api.com</li></a>
                  <a href="https://thesimpsonsquoteapi.glitch.me/"><li>thesimpsonsquoteapi.glitch.me</li></a>
                  <a href="https://www.thecolorapi.com/"><li>www.thecolorapi.com</li></a>
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
              </v-card-actions>
            </v-card>
          </v-col>
          <v-col xs="12" sm="6" md="4" lg="3" xl="2" v-for="(monster, index) in generatedMonsters.slice().reverse()" :key="index">
            <v-card :dark="monster.darkCard" :color="monster.backgroundColor">
              <v-card-title>{{monster.name.fullName}}</v-card-title>
              <v-card-subtitle>{{monster.type}}</v-card-subtitle>
              <v-img :src="monster.roboHashPath"></v-img>
              <v-card-text class="mx-auto black--text" style="background: #FAFAFA; overflow-y: scroll; height:100px">
                <p>
                  {{monster.quote}}
                </p>
              </v-card-text>
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
      const fetchContrastColor = fetch('https://www.thecolorapi.com/id?hex=' + randomColor)
        .then(res => {
          return res.json()
        })
      Promise.all([fetchName, fetchQuote, fetchContrastColor]).then(vals =>{
        this.generatedMonsters.push({
          name: vals[0],
          type: this.monsterTypes[this.selectedMonsterTab].name,
          quote: vals[1][0].quote,
          backgroundColor: '#' + randomColor,
          darkCard: vals[2].contrast.value != '#000000',
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
    }
  },
  mounted() {
    if (localStorage.generatedMonsters) {
      this.generatedMonsters = JSON.parse(localStorage.getItem('generatedMonsters'))
    }
    if (localStorage.currentMonsterType) {
      this.selectedMonsterTab = JSON.parse(localStorage.getItem('selectedMonsterTab'))
    }
    if (this.generatedMonsters.length == 0) {
      this.getMonster()
    }
  }
};
</script>
