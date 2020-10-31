<template>
  <div class="cell">

  </div>
</template>

<script>
    const prefix = `PREFIX : <http://snowman/antoine/erwann#>
        PREFIX swrl: <http://www.semanticweb.org/21504712/ontologies/2020/9/jouer#>`

    export default {
        name: "BigSnowball",
        props: ["hasX", "hasY"],
        computed: {
            getNumber: function () {
                return Number(Number(this.hasX) * 10) + Number(this.hasY)
            }
        },
        methods: {
            getQueryBody: function (dir) {
                return `{
            ?player :${dir} ?cell .
            ?cell :${dir} ?target ;
        swrl:movableOn ?target .
            ?target a :CellFree .
            ?player a :CellPlayer .
        BIND(:CellFree AS ?targettype)
        BIND(:BigSnowball AS ?transform)
      }`
            },
            isAllowedMove: function (dir, executeCallback) {
                return new Promise((resolve, reject) => {
                    let queryString =
                        ` ${prefix}
                     ASK
                     WHERE
                      {
                        ${this.getQueryBody(dir)}
                      }`
                    executeCallback(queryString, true)
                        .then((res) => {
                            resolve(res);
                        }).catch(e => reject(e))
                })
            },
            move: function (targetComp, dir, executeCallback) {
                return new Promise((resolve, reject) => {
                    let queryString =
                        `${prefix}

                    # SmallSnowall
                    DELETE {
                            ?cell a :BigSnowball .
                            ?target a ?targettype .
                    }
                    INSERT {
                            ?cell a :CellFree .
                            ?target a ?transform .
                    }
                    WHERE
                    {
                       ${this.getQueryBody(dir)}
                    }`
                    executeCallback(queryString, true)
                        .then(() => {
                            resolve()
                        }).catch(e => reject(e))
                })
              console.log("BMOOVE");

            }
        }
    }
</script>


<style scoped>
    .cell{
        background-size: cover;
        background-image: url("../../../assets/BigSnowball.jpg");
        text-align: center;
        font-weight: bold;
        width: 60px;
        height: 60px;
        z-index: 1;
    }
</style>
