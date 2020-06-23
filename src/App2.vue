<template>
  <div id="app">
    <header class="page-header">
      <h1 class="page-title">SPORT</h1>
    </header>
    <nav class="nav-bar">
      <div>Football <svg class="chevron" width="31.9" height="32" viewBox="0 0 31.9 32"><path d="M29 16L3 0v7.2L17.6 16 3 24.8V32z"></path></svg> Meow Meow League</div>
    </nav>
    <section class="page-content">
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
        </tr>
        <tr v-for="team in newStandings" v-bind:key="team.Rank" class="standings__team-row">
          <td class="standings__team standings__rank">{{ team.Rank }}</td>
          <td class="standings__team standings__team-name">{{ team.Team }} <span class="standings__manager">{{ team.Manager }}</span></td>
          <td class="standings__team standings__played">{{ team.W + team.D + team.L }}</td>
          <td class="standings__team standings__won">{{ team.W }}</td>
          <td class="standings__team standings__drawn">{{ team.D }}</td>
          <td class="standings__team standings__lost">{{ team.L }}</td>
          <td class="standings__team standings__fpts">{{ team.FPts }}</td>
          <td class="standings__team standings__pts">{{ team.Pts }}</td>
        </tr>
      </table>
    </section>
  </div>
</template>

<script>
const d3 = require('d3')
const _ = require('lodash')
// const numFormat = d3.format('.0f')
// const signedNum = d3.format('+.0f')
// const arrayLast = arr => arr[arr.length - 1]

export default {
  name: 'app',

  components: {
  },

  data: function () {
    return {
      ogStandings: [],
      fixtures: [],
    }
  },

  computed: {
    newStandings: function () {
      const fixtureFilter = d => d.Status === "Complete"
      const fixtures = this.fixtures.filter(fixtureFilter)
      const standings = _.cloneDeep(this.ogStandings)
      const dict = {}
      standings.forEach(s => {
        dict[s.Manager] = s
      })
      fixtures.forEach(f => {
        let home = dict[f.Home]
        let away = dict[f.Away]
        if (f['Home Score'] === f['Away Score']) {
          home.D += 1
          away.D += 1
          home.Pts += 1
          away.Pts += 1
        } else if (f['Home Score'] >= f['Away Score']) {
          home.W += 1
          away.L += 1
          home.Pts += 3
        } else {
          home.L += 1
          away.W += 1
          away.Pts += 3
        }
        home.FPts += f['Home Score']
        away.FPts += f['Away Score']
      })
      standings.sort((a, b) => d3.descending(a.Pts, b.Pts) || d3.descending(a.FPts, b.FPts))
      standings.forEach((d, i) => d.Rank = i + 1)
      return standings
    }
  },

  methods: {
    fetchData: function () {
      const ranges = d3.json(`https://sheets.googleapis.com/v4/spreadsheets/1_-dJPyOSfFklsO3rcHI9V2zMj6B-HksKKceRorSbDQ8/values:batchGet?ranges='Table @ Lockdown'!A1:I&ranges='Fixtures'!A1:F&majorDimension=ROWS&key=AIzaSyB9PQAdV0frspxEE0IUR7yRSBVxhV0LtEQ`)

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
        this.ogStandings = promoteHeaders[0]
        this.fixtures = promoteHeaders[1]
      })
    }
  },

  mounted: function () {
    this.fetchData()
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Maven+Pro:400,500,700&display=swap');

html {
  font-size: 14px;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, avenir next, avenir, helvetica neue, helvetica, Ubuntu, roboto, noto, segoe ui, arial, sans-serif;
  margin: 0;
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

.standings {
  border-collapse: collapse;
  box-sizing: border-box;
  font-weight: 400;
  width: 100%;
}

.standings__header {
  background-color: rgb(242, 242, 240);
  border-bottom: 1px solid rgb(223,223,222);
  font-weight: 600;
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
}

.standings__manager {
  color: rgb(96, 96, 96);
  display: none;
  font-size: .75em;
  font-weight: 400;
  margin-left: .25em;
}

.standings__rank {
  padding-left: .6em;
  text-align: left;
}

.standings__pts {
  font-weight: 700;
  padding-right: .6em;
}

.standings__won,
.standings__drawn,
.standings__lost,
.standings__fpts,
.standings__pts {
  width: 3em;
}

.standings__fpts,
.standings__played {
  display: none;
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
}
</style>
