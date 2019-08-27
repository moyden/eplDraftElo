<template>
  <div id="app">
    <ul class="elo-ranks">
      <li class="elo-ranks__rank elo-ranks__rank--header">
        <div class="elo-ranks__index">

        </div>
        <div class="elo-ranks__team-name">
          Team
        </div>
        <div class="elo-ranks__wins">
          W
        </div>
        <div class="elo-ranks__draws">
          D
        </div>
        <div class="elo-ranks__losses">
          L
        </div>
        <div class="elo-ranks__pts">
          Pts
        </div>
        <div class="elo-ranks__elo">
          ELO
        </div>
      </li>

      <li v-for="(team, index) in eloRanks" class="elo-ranks__rank">
        <div class="elo-ranks__index">
          {{ index + 1 }}
        </div>
        <div class="elo-ranks__team-name">
          {{ team.teamName }}
        </div>
        <div class="elo-ranks__team-owner">
          {{ team.teamOwner }}
        </div>
        <div class="elo-ranks__wins">
          {{ team.actualWins }}
        </div>
        <div class="elo-ranks__draws">
          {{ team.actualDraws }}
        </div>
        <div class="elo-ranks__losses">
          {{ team.actualLosses }}
        </div>
        <div class="elo-ranks__pts">
          {{ team.actualPoints }}
        </div>
        <div class="elo-ranks__elo">
          {{ team.currentElo }}
        </div>
        <div class="elo-ranks__elo-delta">
          {{ team.eloChange }}
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
const signedNum = d3.format('+.0f')
const arrayLast = arr => arr[arr.length - 1]

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
  #app {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  }

  .elo-ranks__rank {
    /* align-items: center; */
    display: grid;
    grid-gap: 0 1rem;
    grid-template-areas:
      "rank name  wins draws losses points elo"
      "rank owner wins draws losses points elo_d";
    grid-template-columns: 3rem 1fr repeat(4, 4rem) 5rem;
    margin: auto;
    padding: .5rem;
    width: 800px;
  }

  .elo-ranks__rank--header {
    font-weight: bold;
  }

  .elo-ranks__index {
    grid-area: rank;
    text-align: center;
  }

  .elo-ranks__team-name {
    grid-area: name;
  }

  .elo-ranks__team-owner {
    align-self: end;
    color: #666;
    font-size: 0.85em;
    grid-area: owner;
  }

  .elo-ranks__wins {
    grid-area: wins;
    text-align: center;
  }

  .elo-ranks__draws {
    grid-area: draws;
    text-align: center;
  }

  .elo-ranks__losses {
    grid-area: losses;
    text-align: center;
  }

  .elo-ranks__pts {
    grid-area: points;
    text-align: center;
  }

  .elo-ranks__elo {
    grid-area: elo;
    text-align: center;
  }

  .elo-ranks__elo-delta {
    align-self: end;
    color: #666;
    font-size: 0.75em;
    grid-area: elo_d;
    text-align: center;
  }
</style>
