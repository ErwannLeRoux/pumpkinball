<template>
  <div class="game-grid">


    <div class="game-cell" v-for="cell in grid" :key="cell.value">
      <component v-if="cell.move" :hasX=cell.x :hasY=cell.y :is="cell.move" :component-type="cell.value" @move-event="move"></component>
      <component v-else :hasX=cell.x :hasY=cell.y :is="cell.value"></component>
    </div>


    <button v-if="allowedLeft" v-on:click='move("hasWeast")'>Left</button>
    <button v-if="allowedRight" v-on:click='move("hasEast")'>Right</button>
    <button v-if="allowedTop" v-on:click='move("hasNorth")'>Top</button>
    <button v-if="allowedBottom" v-on:click='move("hasSouth")'>Bottom</button>
    <button v-on:click='reset(2)'>RESET</button>
  </div>
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
              console.log(this.grid)
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
  .game-grid {
    display: grid;
    grid-template-columns: repeat(10, 100px);
    grid-template-rows: repeat(10, 100px) ;
  }

  .game-cell {

  }
</style>
