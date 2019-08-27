<template>
  <div id="app">
    <ul class="elo-ranks">
      <li v-for="team in eloRanks" class="elo-ranks__rank">
        <div class="elo-ranks__team-name">
          {{ team.teamName }}
        </div>
        <div class="elo-ranks__team-owner">
          {{ team.teamOwner }}
        </div>
        <div class="elo-ranks__elo">
          {{ team.currentElo }}
        </div>
      </li>
    </ul>

    <pre>
      {{ eloRanks }}
    </pre>
  </div>
</template>

<script>
const d3 = require('d3')
const numFormat = d3.format('.0f')
const arrayLast = arr => arr[arr.length - 1]

export default {
  name: 'app',

  components: {
  },

  data: function () {
    return {
      teams: [],
      matches: []
    }
  },

  computed: {
    eloRanks: function () {
      const teams = this.teams.map(t => {
        return {
          teamId: t.id,
          teamName: t.entry_name,
          teamOwner: t.player_first_name,
          elo: [1500]
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
      }

      teams.forEach(t => {
        t.currentElo = numFormat(arrayLast(t.elo))
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
      const r1 = elo1 + k * (s1 - e1)
      const r2 = elo2 + k * (s2 - e2)
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
  .elo-ranks__rank {
    display: grid;
    grid-gap: 1rem;
    grid-template-areas:
      "name elo"
      "owner elo";
    grid-template-columns: 1fr 1fr;
    padding: 1rem;
    width: 600px;
  }

  .elo-ranks__team-name {
    grid-area: name;
  }

  .elo-ranks__team-owner {
    grid-area: owner;
  }

  .elo-ranks__elo {
    grid-area: elo;
  }
</style>
