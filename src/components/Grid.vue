<template>
  <h1 class="game-title">Pumpkin'ball</h1>
  <h2>Choisis le nombre d'hommes citrouilles !</h2>
  <div class="input-container">
    <input v-model="nb" type="number" id="man-number" name="man-number" min="1" max="4">
    <button v-on:click='reset(this.nb)'><span class="go-button">GO !</span></button>
  </div>

  <main>
    <div class="bats"></div>
    <div class="game-grid">
      <div class="game-cell" v-for="(cell, index) in grid" :key="index">
        <component v-if="cell.move" :hasX=cell.x :hasY=cell.y :is="cell.move" :component-type="cell.value" @move-event="move"></component>
        <component v-else :hasX=cell.x :hasY=cell.y :is="cell.value"></component>
      </div>
    </div>
    <div class="bats"></div>
  </main>
</template>

<script>

  const {Connection, query} = require('stardog')
  import CellFree from "./Cells/CellFree.vue";
  import CellPlayer from "./Cells/CellPlayer.vue";
  import BigSnowball from "./Cells/SimpleSnowballs/BigSnowball.vue";
  import MediumSnowball from "./Cells/SimpleSnowballs/MediumSnowball.vue";
  import SmallSnowball from "./Cells/SimpleSnowballs/SmallSnowball.vue";
  import MediumAndBig from "./Cells/ComposedSnowballs/MediumAndBig.vue";
  import SmallAndBig from "./Cells/ComposedSnowballs/SmallAndBig.vue";
  import SmallAndMedium from "./Cells/ComposedSnowballs/SmallAndMedium.vue";
  import SmallAndMediumAndBig from "./Cells/ComposedSnowballs/SmallAndMediumAndBig.vue";
  import CellNorthPlayer from "./Cells/MoveCell/CellNorthPlayer";
  import CellSouthPlayer from "./Cells/MoveCell/CellSouthPlayer";
  import CellEastPlayer from "./Cells/MoveCell/CellEastPlayer";
  import CellWestPlayer from "./Cells/MoveCell/CellWestPlayer";


  export default {
    name: 'Grid',
    data: function () {
      return {
        grid: [],
        nb: 1,
        conn: null,
        allowedTop: false,
        allowedBottom: false,
        allowedLeft: false,
        allowedRight: false,
      }
    },
    components: {
      'CellFree': CellFree,
      'CellPlayer': CellPlayer,
      'SmallSnowball': SmallSnowball,
      'MediumSnowball': MediumSnowball,
      'BigSnowball': BigSnowball,
      'MediumAndBig': MediumAndBig,
      'SmallAndBig': SmallAndBig,
      'SmallAndMedium': SmallAndMedium,
      'SmallAndMediumAndBig': SmallAndMediumAndBig,
      'CellNorthPlayer': CellNorthPlayer,
      'CellSouthPlayer': CellSouthPlayer,
      'CellEastPlayer' : CellEastPlayer,
      'CellWestPlayer' : CellWestPlayer,
    },
    methods: {
      reset: function (nbSnowman) {
        let ClearQueryString = `
        PREFIX : <http://snowman/antoine/erwann#>
        DELETE{
            ?cell a ?type
        }
        INSERT{
            ?cell a :CellFree
        }
        WHERE{
            ?cell a ?type
        filter(
            ?type = :CellPlayer ||
            ?type = :SmallSnowball ||
            ?type = :MediumSnowball ||
            ?type = :BigSnowball ||
            ?type = :SmallAndBig ||
            ?type = :SmallAndMedium ||
            ?type = :MediumAndBig ||
            ?type = :SmallAndMediumAndBig
        )}
      `

        let copy = Array.from(this.grid)
        copy = copy.filter(function(cell){
          return (cell.x != 0 && cell.x !=9 && cell.y!=0 && cell.y!=9);
        });
        copy.sort(() => 0.5 - Math.random());
        let selected = copy.slice(0, nbSnowman*3+1);
        selected[0].type = ':CellPlayer';
        for( let i = 0; i < nbSnowman;i++) {
          selected[i*3+1].type = ':SmallSnowball';
          selected[i*3+2].type = ':MediumSnowball';
          selected[i*3+3].type = ':BigSnowball';
        }

        let insertBody='';
        let deleteBody=''
        selected.forEach( cell =>{
          insertBody+=`:cell${cell.x.concat(cell.y)} a ${cell.type}. `;
          deleteBody+=`:cell${cell.x.concat(cell.y)} a :CellFree. `
        })
        let generateQueryString = `
        PREFIX : <http://snowman/antoine/erwann#>
        DELETE DATA{
          ${deleteBody}
        };
        INSERT DATA{
          ${insertBody}
        }
      `
        this.execute(ClearQueryString, false).then( ()=>{
          this.execute(generateQueryString, false).then( () => {
            this.update();
          });
        });

      },
      update: function () {

        this.execute(
            `PREFIX : <http://snowman/antoine/erwann#>
           SELECT DISTINCT *
            WHERE {
                ?c a ?concept.
                ?c :hasCoordonneeX ?x .
                ?c :hasCoordonneeY ?y .
                OPTIONAL{
                    ?c a ?moveConcept.
                    FILTER(?moveConcept in (:CellNorthPlayer, :CellSouthPlayer ,:CellEastPlayer,:CellWestPlayer))
                }
                FILTER (
                    ?concept in (:CellFree, :CellPlayer, :SmallSnowball, :MediumSnowball, :BigSnowball, :MediumAndBig, :SmallAndBig, :SmallAndMedium, :SmallAndMediumAndBig)
                )
            } ORDER BY ?c`, true)
            .then(data => {
              this.grid = []
              data.forEach(item => {
                this.grid.push({
                  value: item.concept.value.split("#")[1],
                  move: item?.moveConcept?.value?.split("#")[1],
                  x: item.x.value,
                  y: item.y.value
                })
              })
            });

      },
      move: function (params) {
        let direction = params[0];
        let compType = params[1];
        this.$options.components[compType].methods.move(compType, direction, this.execute).then(() => {
          this.$options.components["CellPlayer"].methods.move(compType, direction, this.execute).then(() => {
            this.update()
          })
        })
      },
      execute: function (queryString, reasoningOn) {
        return new Promise((resolve, reject) => {
          query.execute(this.conn,
              process.env.VUE_APP_DATABASE,
              queryString,
              'application/sparql-results+json',
              {
                offset: 0,
                reasoning: reasoningOn
              }
          ).then(({body}) => {
            if (body && body.results && body.results.bindings) {
              resolve(body.results.bindings)
            } else if (body && (body.boolean == false || body.boolean == true)) {
              resolve(body.boolean);
            } else {
              resolve("no result provided");
            }
          }).catch(e => {
            reject(e)
          })
        })
      },
    },
    mounted() {
      this.conn = new Connection({
        username: 'admin',
        password: 'admin',
        endpoint: 'http://192.168.56.103:5820',
      });
      this.update()
    }
  }

</script>

<style scoped>
  @font-face {
    font-family: 'Creepster';
    font-style: normal;
    font-weight: 400;
    src: local('Creepster'), local('Creepster-Regular'), url(https://fonts.gstatic.com/s/creepster/v9/AlZy_zVUqJz4yMrniH4Rcn38.ttf) format('truetype');
  }
  @font-face {
    font-family: 'Lato';
    font-style: normal;
    font-weight: 900;
    src: local('Lato Black'), local('Lato-Black'), url(https://fonts.gstatic.com/s/lato/v17/S6u9w4BMUTPHh50XSwiPHA.ttf) format('truetype');
  }


  .game-grid {
    display: grid;
    grid-template-columns: repeat(10, 60px);
    grid-template-rows: repeat(10, 60px) ;
    justify-content: center;
    margin: 0 50px;
  }

  h1, h2, .go-button {
    background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiA/PjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIiB2aWV3Qm94PSIwIDAgMSAxIiBwcmVzZXJ2ZUFzcGVjdFJhdGlvPSJub25lIj48bGluZWFyR3JhZGllbnQgaWQ9Imxlc3NoYXQtZ2VuZXJhdGVkIiBncmFkaWVudFVuaXRzPSJ1c2VyU3BhY2VPblVzZSIgeDE9IjAlIiB5MT0iMCUiIHgyPSIwJSIgeTI9IjEwMCUiPjxzdG9wIG9mZnNldD0iMCUiIHN0b3AtY29sb3I9IiMyYzNlNTAiIHN0b3Atb3BhY2l0eT0iMSIvPjxzdG9wIG9mZnNldD0iMTAwJSIgc3RvcC1jb2xvcj0iIzhlNDRhZCIgc3RvcC1vcGFjaXR5PSIxIi8+PC9saW5lYXJHcmFkaWVudD48cmVjdCB4PSIwIiB5PSIwIiB3aWR0aD0iMSIgaGVpZ2h0PSIxIiBmaWxsPSJ1cmwoI2xlc3NoYXQtZ2VuZXJhdGVkKSIgLz48L3N2Zz4=);
    background-image: -webkit-linear-gradient(top, #2c3e50 0%, #8e44ad 100%);
    background-image: -moz-linear-gradient(top, #2c3e50 0%, #8e44ad 100%);
    background-image: -o-linear-gradient(top, #2c3e50 0%, #8e44ad 100%);
    background-image: linear-gradient(to bottom, #2c3e50 0%, #8e44ad 100%);
    -webkit-text-fill-color: transparent;
    -webkit-background-clip: text;
    margin: 20px auto;
    text-align: center;
    color: #fff;
  }

  main {
    display: flex;
    justify-content: center;
  }

  h2 {
    font: 100 30px Creepster, Helvetica, sans-serif;
  }

  h1 {
    font: 100 120px Creepster, Helvetica, sans-serif;
  }

  .go-button {
    font: 100 20px Creepster, Helvetica, sans-serif;
  }

  .bats {
    margin-top: 30px;
    height: 550px;
    width: 200px;
    background-image: url("../assets/bats.png");
  }

  .input-container {
    width: 100%;
    text-align: center;
    margin-bottom: 20px;
  }
</style>
