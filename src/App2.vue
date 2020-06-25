<template>
  <div id="app">
    <header class="page-header">
      <h1 class="page-title">SPORT</h1>
    </header>
    <nav class="nav-bar">
      <div>Football <svg class="chevron" width="31.9" height="32" viewBox="0 0 31.9 32"><path d="M29 16L3 0v7.2L17.6 16 3 24.8V32z"></path></svg> Meow Meow League</div>
    </nav>
    <section class="page-content">
      <div @click="toggleAsItStands" class="as-it-stands" :class="{ 'as-it-stands--active': asItStands }">
        As It Stands
      </div>
      <table class="standings">
        <tr class="standings__header-row--desktop">
          <th class="standings__header standings__rank"></th>
          <th class="standings__header standings__team-name">Team</th>
          <th class="standings__header standings__played">Played</th>
          <th class="standings__header standings__won">Won</th>
          <th class="standings__header standings__drawn">Drawn</th>
          <th class="standings__header standings__lost">Lost</th>
          <th class="standings__header standings__fpts">+</th>
          <th class="standings__header standings__pts">Points</th>
          <th class="standings__header standings__form">Form</th>
        </tr>
        <tr class="standings__header-row--mobile">
          <th class="standings__header standings__rank"></th>
          <th class="standings__header standings__team-name">Team</th>
          <th class="standings__header standings__played">P</th>
          <th class="standings__header standings__won">W</th>
          <th class="standings__header standings__drawn">D</th>
          <th class="standings__header standings__lost">L</th>
          <th class="standings__header standings__fpts">+</th>
          <th class="standings__header standings__pts">Pts</th>
          <th class="standings__header standings__form">Fm</th>
        </tr>
        <tr v-for="team in newStandings" v-bind:key="team.Rank" class="standings__team-row">
          <td class="standings__team standings__rank">{{ team.Rank }}</td>
          <td class="standings__team standings__team-name">
            <a class="standings__team-link" :href="`https://draft.premierleague.com/entry/${team.id}/event/${activeGameweek}`" target="_blank">{{ team.Team }}</a>
            <span class="standings__manager">
              {{ team.Manager }}
            </span>
          </td>
          <td class="standings__team standings__played">{{ team.W + team.D + team.L }}</td>
          <td class="standings__team standings__won">{{ team.W }}</td>
          <td class="standings__team standings__drawn">{{ team.D }}</td>
          <td class="standings__team standings__lost">{{ team.L }}</td>
          <td class="standings__team standings__fpts">{{ team.FPts }}</td>
          <td class="standings__team standings__pts">{{ team.Pts }}</td>
          <td class="standings__team standings__form">
            <span v-for="result in team.Form" :class="[`form__result`, `form__result--${result}`]">{{ result }}</span>
          </td>
        </tr>
      </table>

      <div class="matches-switch">
        <div @click="showScores" class="matches-switch__scores" :class="[ fixturesActive ? `` : `matches-switch__scores--active`]">
          Scores & Results
        </div>
        <div @click="showFixtures" class="matches-switch__fixtures" :class="[ fixturesActive ? `matches-switch__fixtures--active` : ``]">
          Fixtures
        </div>
      </div>

      <div class="scores" :class="[ fixturesActive ? `` : `scores--active`]">
        <ol class="scores__gameweeks">
          <li v-for="gameweek in scores" class="scores__gameweek">
            <div class="gameweek__header">
              Gameweek {{ gameweek.gameweek }}
            </div>
            <ul class="gameweek__matches">
              <li v-for="match in gameweek.games" class="gameweek__match" :class="[`gameweek__match--${match.Status}`]">
                <div class="match__home-team">
                  <a :href="`https://draft.premierleague.com/entry/${match['Home Id']}/event/${gameweek.gameweek + 9}`" target="_blank">
                    {{ match.homeTeam }}
                  </a>
                </div>
                <div class="match__home-manager">{{ match.Home }}</div>
                <div class="match__home-score">{{ match['Home Score'] }}</div>
                <div class="match__away-team">
                  <a :href="`https://draft.premierleague.com/entry/${match['Away Id']}/event/${gameweek.gameweek + 9}`" target="_blank">
                    {{ match.awayTeam }}
                  </a>
                </div>
                <div class="match__away-manager">{{ match.Away }}</div>
                <div class="match__away-score">{{ match['Away Score'] }}</div>
                <div class="match__status">{{ match.Status === 'Complete' ? 'FT' : '' }}</div>
              </li>
            </ul>
          </li>
        </ol>
      </div>

      <div class="fixtures" :class="[ fixturesActive ? `fixtures--active` : ``]">
        <ol class="fixtures__gameweeks">
          <li v-for="gameweek in fixtures" class="fixtures__gameweek">
            <div class="gameweek__header">
              Gameweek {{ gameweek.gameweek }}
            </div>
            <ul class="gameweek__matches">
              <li v-for="match in gameweek.games" class="gameweek__match" :class="[`gameweek__match--${match.Status}`]">
                <div class="match__home-team">{{ match.homeTeam }}</div>
                <div class="match__home-manager">{{ match.Home }}</div>
                <div class="match__home-score">{{ match['Home Score'] }}</div>
                <div class="match__away-team">{{ match.awayTeam }}</div>
                <div class="match__away-manager">{{ match.Away }}</div>
                <div class="match__away-score">{{ match['Away Score'] }}</div>
                <div class="match__status">{{ match.Status === 'Complete' ? 'FT' : '' }}</div>
              </li>
            </ul>
          </li>
        </ol>
      </div>
    </section>
  </div>
</template>

<script>
const d3 = require('d3')
const _ = require('lodash')

export default {
  name: 'app',

  components: {
  },

  data: function () {
    return {
      ogStandings: [],
      matches: [],
      activeGameweek: null,
      fixturesActive: false,
      asItStands: true
    }
  },

  computed: {
    newStandings: function () {
      let fixtureFilter
      if (this.asItStands) fixtureFilter = d => d.Status === "Complete" || d.Status === "Active"
      else fixtureFilter = d => d.Status === "Complete"
      const fixtures = this.matches.filter(fixtureFilter)
      const standings = _.cloneDeep(this.ogStandings)
      const dict = {}
      standings.forEach(s => {
        dict[s.Manager] = s
        s.Form = s.Form.split('')
      })
      fixtures.forEach(f => {
        let home = dict[f.Home]
        let away = dict[f.Away]
        if (f['Home Score'] === f['Away Score']) {
          home.D += 1
          away.D += 1
          home.Pts += 1
          away.Pts += 1
          home.Form = [...home.Form.slice(1, home.Form.length), 'D']
          away.Form = [...away.Form.slice(1, away.Form.length), 'D']
        } else if (f['Home Score'] >= f['Away Score']) {
          home.W += 1
          away.L += 1
          home.Pts += 3
          home.Form = [...home.Form.slice(1, home.Form.length), 'W']
          away.Form = [...away.Form.slice(1, away.Form.length), 'L']
        } else {
          home.L += 1
          away.W += 1
          away.Pts += 3
          home.Form = [...home.Form.slice(1, home.Form.length), 'L']
          away.Form = [...away.Form.slice(1, away.Form.length), 'W']
        }
        home.FPts += f['Home Score']
        away.FPts += f['Away Score']
      })
      standings.sort((a, b) => d3.descending(a.Pts, b.Pts) || d3.descending(a.FPts, b.FPts))
      standings.forEach((d, i) => d.Rank = i + 1)
      return standings
    },

    fixtures: function () {
      const matches = this.matches.filter(m => m.Status === 'Scheduled')
      const gameweeks = new Set(matches.map(m => m.Gameweek))
      return [...gameweeks].map(gw => {
        return {
          gameweek: gw,
          games: matches.filter(m => m.Gameweek === gw)
        }
      })
    },

    scores: function () {
      const matches = this.matches.filter(m => m.Status === 'Active' || m.Status === 'Complete')
      const gameweeks = [...new Set(matches.map(m => m.Gameweek))].sort((a, b) => d3.descending(a, b))
      return gameweeks.map(gw => {
        return {
          gameweek: gw,
          games: matches.filter(m => m.Gameweek === gw)
        }
      })
    }
  },

  methods: {
    fetchData: function () {
      const ranges = d3.json(`https://sheets.googleapis.com/v4/spreadsheets/1_-dJPyOSfFklsO3rcHI9V2zMj6B-HksKKceRorSbDQ8/values:batchGet?ranges='Table @ Lockdown'!A1:K&ranges='Fixtures'!A1:H&ranges='Gameweek'!A1:B&majorDimension=ROWS&key=AIzaSyB9PQAdV0frspxEE0IUR7yRSBVxhV0LtEQ`)

      ranges.then(data => {
        const promoteHeaders = data.valueRanges.map(range => {
          const header = range.values[0]
          const body = range.values.slice(1, range.values.length).map(d => {
            const obj = {}
            header.forEach((h, i) => {
              obj[h] = d[i]
            })
            for (let key of Object.keys(obj)) {
              if (!isNaN(+obj[key])) obj[key] = +obj[key]
            }
            return obj
          })
          return body
        })
        const managers = promoteHeaders[0].reduce((a, c) => {
          a[c.Manager] = c.Team
          return a
        }, {})
        promoteHeaders[1].forEach(d => {
          d.homeTeam = managers[d.Home]
          d.awayTeam = managers[d.Away]
        })
        window.console.log(managers)
        this.ogStandings = promoteHeaders[0]
        this.matches = promoteHeaders[1]
        this.activeGameweek = promoteHeaders[2][promoteHeaders[2].length - 1].Gameweek
      })
    },

    showFixtures: function () {
      this.fixturesActive = true
    },

    showScores: function () {
      this.fixturesActive = false
    },

    toggleAsItStands: function () {
      this.asItStands = !this.asItStands
    }
  },

  mounted: function () {
    this.fetchData()

    setInterval(function () {
      this.fetchData()
    }.bind(this), 120000)
  }
}
</script>

<style>
html {
  font-size: 14px;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, avenir next, avenir, helvetica neue, helvetica, Ubuntu, roboto, noto, segoe ui, arial, sans-serif;
  margin: 0;
}

a {
  color: rgb(0, 0, 0);
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

.page-header {
  background-color: rgb(255, 210, 48);
  padding: 12px 16px;
}

.nav-bar {
  background-color: black;
  color: white;
  padding: 6px 16px;
}

.nav-bar > div {
  display: block;
  margin: auto;
  max-width: 800px;
  width: 100%;
}

.chevron {
  fill: white;
  height: 12px;
  width: 12px;
}

.page-title {
  font-size: 32px;
  font-weight: 700;
  margin: auto;
  padding: 0;
  max-width: 800px;
  width: 100%;
}

.page-content {
  margin: auto;
  max-width: 800px;
  padding: 1rem;
}

.as-it-stands {
  background-color: rgb(219, 219, 219);
  color: rgb(90, 90, 90);
  cursor: pointer;
  display: inline-block;
  font-size: .85em;
  font-weight: 700;
  margin-bottom: 1rem;
  padding: .25em .5em;
  text-transform: uppercase;
  user-select: none;
}

.as-it-stands--active {
  background-color: rgb(40, 102, 246);
  color: rgb(255, 255, 255);
}

.standings {
  border-collapse: collapse;
  box-sizing: border-box;
  font-weight: 400;
  width: 100%;
}

.standings__header {
  background-color: rgb(242, 242, 240);
  border-bottom: 1px solid rgb(223,223,222);
  font-weight: 700;
  text-align: right;
  padding: 1.15em .3em .3em;
}

.standings__header-row--desktop {
  display: none;
}

.standings__team {
  border-bottom: 1px solid rgb(233, 233, 232);
  line-height: 1em;
  padding: .6em .3em;
  text-align: right;
}

.standings__team-row:nth-child(3) .standings__team,
.standings__team-row:nth-child(9) .standings__team {
  border-bottom: 1px dashed rgb(0, 0, 0);
}

.standings__team-name {
  font-weight: 700;
  text-align: left;
  vertical-align: middle;
}

.standings__team-link {
  vertical-align: middle;
}

.standings__manager {
  color: rgb(96, 96, 96);
  display: none;
  font-size: .75em;
  font-weight: 400;
  margin-left: .25em;
}

.standings__rank {
  padding-left: 1em;
  text-align: left;
}

.standings__pts {
  font-weight: 700;
  padding-right: 1em;
}

.standings__form {
  border-left: 1px solid rgb(233, 233, 232);
  display: none;
  padding-left: 1em;
  text-align: left;
}

.standings__won,
.standings__drawn,
.standings__lost,
.standings__fpts,
.standings__pts {
  width: 2.75em;
}

.standings__fpts,
.standings__played {
  display: none;
}

.form__result {
  background-color: black;
  color: white;
  display: inline-block;
  font-size: .8em;
  height: 16px;
  line-height: 16px;
  margin-right: 2px;
  text-align: center;
  vertical-align: center;
  width: 16px;
}

.form__result--W { background-color: rgb(68, 196, 36) }
.form__result--L { background-color: rgb(221, 0, 0) }
.form__result--D { background-color: rgb(90, 90, 90) }

.matches-switch {
  border: 1px solid rgb(233, 233, 232);
  display: grid;
  grid-template-columns: 1fr 1fr;
  margin: 2rem 0 1rem;
}

.matches-switch__scores,
.matches-switch__fixtures {
  cursor: pointer;
  padding: 1em;
  text-align: center;
  text-transform: uppercase;
  user-select: none;
}

.matches-switch__scores--active,
.matches-switch__fixtures--active {
  background-color: rgb(64, 64, 64);
  color: rgb(255, 255, 255);
}

.scores,
.fixtures {
  display: none;
  margin-top: 1rem;
}

.scores--active,
.fixtures--active {
  display: block;
}

.scores__gameweeks,
.fixtures__gameweeks,
.gameweek__matches {
  list-style: none;
  padding: 0;
}

.gameweek__header {
  background-color: rgb(242, 242, 240);
  font-size: .85em;
  padding: .6em;
  text-transform: uppercase;
}

.gameweek__match {
  border-bottom: 1px solid rgb(233, 233, 232);
  display: grid;
  font-size: .9rem;
  grid-gap: 1px;
  grid-template-areas: "home-team    home-score away-score away-team"
                       "home-manager status     status     away-manager";
  grid-template-columns: 1fr 2.125em 2.125em 1fr;
  margin: 1em 0;
  padding-bottom: .5em;
}

.gameweek__match:last-child {
  border-bottom: none;
}

.match__home-team { grid-area: home-team; }
.match__home-manager { grid-area: home-manager; }
.match__home-score { grid-area: home-score; }
.match__away-team { grid-area: away-team; }
.match__away-manager { grid-area: away-manager; }
.match__away-score { grid-area: away-score; }
.match__status { grid-area: status; }

.match__home-team,
.match__home-manager {
  text-align: right;
}

.match__home-team,
.match__away-team {
  font-weight: 700;
  line-height: 2.125em;
  padding: 0 1rem;
}

.match__home-score,
.match__away-score {
  background-color: rgb(255, 210, 48);
  font-weight: 700;
  line-height: 2.125em;
  max-height: 2.125em;
  text-align: center;
}

.gameweek__match--Active .match__home-score,
.gameweek__match--Active .match__away-score {
  background-color: rgb(40, 102, 246);
  color: rgb(255, 255, 255);
}

.gameweek__match--Scheduled .match__home-score,
.gameweek__match--Scheduled .match__away-score {
  background-color: rgb(219, 219, 219);
  color: rgb(90, 90, 90);
}

.match__home-manager,
.match__away-manager {
  color: rgb(96, 96, 96);
  font-size: .85em;
  line-height: 2.125em;
  padding: 0 1rem;
}

.match__status {
  color: rgb(96, 96, 96);
  font-size: .85em;
  line-height: 2.125em;
  text-align: center;
}

@media all and (min-width: 480px) {
  .standings__manager {
    display: inline;
  }
}

@media all and (min-width: 600px) {
  .standings__header-row--desktop {
    display: table-row;
  }

  .standings__header-row--mobile {
    display: none;
  }

  .standings__rank {
    width: 1.5em;
  }

  .standings__played,
  .standings__won,
  .standings__drawn,
  .standings__lost,
  .standings__fpts,
  .standings__pts {
    width: 9%;
  }

  .standings__fpts,
  .standings__played {
    display: table-cell;
  }

  .gameweek__match {
    font-size: 1rem;
  }
}

@media all and (min-width: 800px) {
  .standings__form {
    display: table-cell;
  }
}
</style>
