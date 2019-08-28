<template>
  <div id="app">
    <div class="ranks">
      <div class="ranks__ddl-sport">
        <span class="ddl-sport__ddl">
          <span class="ddl-sport__square">D</span>
          <span class="ddl-sport__square">D</span>
          <span class="ddl-sport__square">L</span>
        </span>
        <span class="ddl-sport__sport">Sport</span>
      </div>

      <div class="ranks__title">
        Meow Meow League
      </div>

      <div class="ranks__header">
        <div class="ranks__rank-title">
          Top
        </div>
        <div class="ranks__team-name-title">

        </div>
        <div class="ranks__wins-title">
          W
        </div>
        <div class="ranks__draws-title">
          D
        </div>
        <div class="ranks__losses-title">
          L
        </div>
        <div class="ranks__points-title">
          Pts
        </div>
        <div class="ranks__elo-title">
          ELO
        </div>
      </div>

      <ul class="ranks__table">
        <li v-for="(team, index) in eloRanks" class="ranks__row">
          <div class="ranks__rank">
            {{ index + 1 }}
          </div>
          <div class="ranks__team-name">
            {{ team.teamName }}
            <span class="ranks__team-owner">{{ team.teamOwner }}</span>
          </div>
          <div class="ranks__wins">
            {{ team.actualWins }}
          </div>
          <div class="ranks__draws">
            {{ team.actualDraws }}
          </div>
          <div class="ranks__losses">
            {{ team.actualLosses }}
          </div>
          <div class="ranks__points">
            {{ team.actualPoints }}
          </div>
          <div class="ranks__elo">
            {{ team.currentElo }}
          </div>
        </li>
      </ul>

      <div class="ranks__hashtag">
        #meowmeowfootball
      </div>

      <div class="ranks__url">
        <a href="https://draft.premierleague.com/league" target="_blank">premierleague.com/meowmeow</a>
      </div>
    </div>

    <div class="left-shadow">

    </div>
  </div>
</template>

<script>
const d3 = require('d3')
const _ = require('lodash')
const numFormat = d3.format('.0f')
const signedNum = d3.format('+.0f')
const arrayLast = arr => arr[arr.length - 1]

const bgs = ['I9HyW30buuQ', '-kML7I_SWhg', 'ObhCU6Vhoe8', '8-s5QuUBtyM', 'a-z2yFMFL74', 'jqUUBHKkwF4', 'MOnU_o4DMQw']
const choice = _.sample(bgs)
d3.select('body').style('background-image', `url('https://source.unsplash.com/${choice}/1600x900')`)

export default {
  name: 'app',

  components: {
  },

  data: function () {
    return {
      teams: [],
      matches: [],
      standings: []
    }
  },

  computed: {
    eloRanks: function () {
      const teams = this.teams.map(t => {
        return {
          teamId: t.id,
          teamName: t.entry_name,
          teamOwner: t.player_first_name,
          elo: [1500],
          actualWins: 0,
          actualDraws: 0,
          actualLosses: 0
        }
      })
      const finishedMatches = this.matches.filter(m => m.finished)

      for (let match of finishedMatches) {
        const home = teams.find(t => t.teamId === match.league_entry_1)
        const away = teams.find(t => t.teamId === match.league_entry_2)
        const pointsDiff = match.league_entry_1_points - match.league_entry_2_points
        let result = 0.5
        if (pointsDiff > 0) result = 1
        if (pointsDiff < 0) result = 0
        this.updateElo(home, away, result)
        if (result === 1) {
          home.actualWins += 1
          away.actualLosses += 1
        } else if (result === 0) {
          home.actualLosses += 1
          away.actualWins += 1
        } else {
          home.actualDraws += 1
          away.actualDraws += 1
        }
      }

      teams.forEach(t => {
        t.currentElo = numFormat(arrayLast(t.elo))
        t.eloChange = signedNum(t.elo[t.elo.length - 1] - t.elo[t.elo.length - 2])
        t.actualPoints = (t.actualWins * 3) + t.actualDraws
      })

      return teams.sort((a, b) => d3.descending(a.currentElo, b.currentElo))
    }
  },

  methods: {
    fetchData: function () {
      const request = d3.json('https://yacdn.org/proxy/https://draft.premierleague.com/api/league/33927/details')
      request.then(data => {
        this.teams = data.league_entries
        this.matches = data.matches
        this.standings = data.standings
      })
    },

    updateElo: function (home, away, result) {
      const elo1 = arrayLast(home.elo)
      const elo2 = arrayLast(away.elo)
      const q1 = Math.pow(10, elo1 / 400)
      const q2 = Math.pow(10, elo2 / 400)
      const e1 = q1 / (q1 + q2)
      const e2 = q2 / (q1 + q2)
      const k = 32
      let s1 = result
      let s2 = 1 - result
      const r1 = Math.round(elo1 + k * (s1 - e1))
      const r2 = Math.round(elo2 + k * (s2 - e2))
      home.elo.push(r1)
      away.elo.push(r2)
    }
  },

  mounted: function () {
    this.fetchData()
  }
}
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Maven+Pro:400,500,700,900&display=swap');

  body {
    /* background-image: url('https://source.unsplash.com/MOnU_o4DMQw/1600x900'); */
    margin: 0;
  }

  #app {
    align-items: center;
    display: grid;
    font-family: 'Maven Pro', sans-serif;
    grid-template-columns: repeat(4, 1fr);
    height: 100vh;
    justify-content: center;
    margin: auto;
    width: 100vw;
  }

  .ranks {
    display: grid;
    grid-column: 2 / 4;
    grid-template-columns: 1fr 1fr;
    position: relative;
  }

  .ranks__ddl-sport {
    background-color: rgb(252, 208, 46);
    bottom: 0;
    content: '';
    display: grid;
    font-weight: 600;
    grid-template-columns: 1fr auto 1fr;
    grid-template-rows: 1fr auto auto 1fr;
    height: calc(8rem - 4px);
    position: absolute;
    right: 100%;
    text-align: center;
    width: calc(8.5rem - 4px);
  }

  .ddl-sport__square {
    background-color: black;
    color: rgb(252, 208, 46);
    display: inline-block;
    font-size: 0.8rem;
    font-weight: 500;
    line-height: 1rem;
    margin: .075rem;
    padding: 0 .025rem .05rem;
    width: 1rem;
  }

  .ddl-sport__ddl {
    grid-column: 2 / 3;
    grid-row: 2 / 3;
  }

  .ddl-sport__sport {
    display: block;
    font-size: 1.5rem;
    grid-column: 2 / 3;
    grid-row: 3 / 4;
    text-transform: uppercase;
  }

  .ranks__title {
    background-color: rgb(252, 208, 46);
    border-color: rgb(253, 226, 136) transparent rgb(239, 199, 82);
    border-style: solid;
    border-width: 2px 0;
    font-size: 1.8em;
    font-weight: 600;
    grid-column: span 2;
    padding: .4em;
    text-transform: uppercase;
  }

  .ranks__header {
    background-color: rgb(218, 214, 200);
    border-color: rgb(240, 238, 236) transparent rgb(203, 200, 192);
    border-style: solid;
    border-width: 1px 0;
    display: grid;
    grid-column: span 2;
    grid-template-columns: 3rem 1fr repeat(4, 3rem) 3.5rem;
    padding: .4em 0;
    text-align: center;
  }

  .ranks__table {
    grid-column: span 2;
    margin: 0;
    padding: 0;
  }

  .ranks__row {
    background-color: rgb(246, 247, 248);
    display: grid;
    grid-template-columns: 3rem 1fr repeat(4, 3rem) 3.5rem;
    text-align: center;
  }

  .ranks__row:nth-child(even) {
    background-color: rgb(241, 242, 243);
  }

  .ranks__row > div {
    padding: .4rem 0;
  }

  .ranks__rank {
    font-weight: 500;
  }

  .ranks__team-name {
    font-weight: 500;
    text-align: left;
    text-transform: uppercase;
  }

  .ranks__team-owner {
    color: #666;
    font-size: .75em;
    font-weight: 400;
    margin-left: .25em;
    text-transform: none;
  }

  .ranks__row:hover .ranks__team-owner {
    color: #666;
  }

  .ranks__wins,
  .ranks__draws,
  .ranks__losses,
  .ranks__points,
  .ranks__elo {
    box-shadow: inset .5rem 0 .8rem -.5rem rgb(217, 217, 217);
  }

  .ranks__points {
    font-weight: 500;
  }

  .ranks__hashtag {
    background-color: rgb(252, 208, 46);
    border-color: rgb(253, 226, 136) transparent rgb(239, 199, 82);
    border-style: solid;
    border-width: 1px 0;
    font-size: .8em;
    padding: .4rem .6rem;
  }

  .ranks__url {
    background-color: rgb(252, 208, 46);
    border-color: rgb(253, 226, 136) transparent rgb(239, 199, 82);
    border-style: solid;
    border-width: 1px 0;
    font-size: .8em;
    padding: .4rem .6rem;
    text-align: right;
  }

  .ranks__url > a {
    color: black;
    text-decoration: none;
  }

  .left-shadow {
    align-self: stretch;
    background-color: transparent;
    box-shadow: inset -2rem 0 3.2rem -2rem rgba(0, 0, 0, 0.5);
    grid-column: 1 / 2;
    grid-row: 1 / 2;
    height: 100%;
    position: absolute;
    width: 25%;
  }
</style>
