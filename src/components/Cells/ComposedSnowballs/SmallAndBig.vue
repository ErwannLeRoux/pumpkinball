<template>
  <div class="cell">

  </div>
</template>

<script>
    const prefix = `PREFIX : <http://snowman/antoine/erwann#>
        PREFIX swrl: <http://www.semanticweb.org/21504712/ontologies/2020/9/jouer#>`

    export default {
        name: "SmallAndBig",
        props: ["hasX", "hasY"],
        computed: {
            getNumber: function () {
                return Number(Number(this.hasX) * 10) + Number(this.hasY)
            }
        },
        methods: {
            getQueryBody: function (dir) {
                return `
        {
            ?player :${dir} ?cell;
                    a :CellPlayer.
            ?cell :${dir} ?target.
            ?target a :CellFree.
            BIND(:CellFree AS ?targettype).
            BIND(:SmallSnowball AS ?transform).
        }
        UNION
        {
            ?player :${dir} ?cell;
                    a :CellPlayer.
            ?cell :${dir} ?target.
            ?target a :MediumAndBig.
            BIND(:MediumAndBig AS ?targettype).
            BIND(:SmallAndMediumAndBig AS ?transform).
        }
        UNION
        {
            ?player :${dir} ?cell;
                    a :CellPlayer.
            ?cell :${dir} ?target.
            ?target a :MediumSnowball.
            BIND(:MediumSnowball AS ?targettype).
            BIND(:SmallAndMedium AS ?transform).
        }
        UNION
        {
            ?player :${dir} ?cell;
                    a :CellPlayer.
            ?cell :${dir} ?target.
            ?target a :BigSnowball.
            BIND(:BigSnowball AS ?targettype).
            BIND(:SmallAndBig AS ?transform).
        }`
            },
            isAllowedMove: function (dir, executeCallback) {
                return new Promise((resolve, reject) => {
                    let queryString =
                        `  ${prefix}
                     ASK
                     WHERE
                      {
                        ${this.getQueryBody(dir)}
                      }`
                    executeCallback(queryString, false)
                        .then((res) => {
                            resolve(res);
                        }).catch(e => reject(e))
                })
            },
            move: function (targetComp, dir, executeCallback) {
                return new Promise((resolve, reject) => {
                    let queryString =
                        `${prefix}

                            #SmallAndBig
                            DELETE{
                                ?cell a :SmallAndBig.
                                ?target a ?targettype
                            }
                            INSERT{
                                ?cell a :BigSnowball.
                                ?target a ?transform
                            }
                            WHERE
                            {
                                ${this.getQueryBody(dir)}
                            }`
                    executeCallback(queryString, false)
                        .then(() => {
                            resolve()
                        }).catch(e => reject(e))
                })
              console.log("SBMOOVE");
            }
        }
    }
</script>

<style scoped>
    .cell {
        background-image: url("../../../assets/BigAndSmall.jpg");
        background-size: cover;
        text-align: center;
        font-weight: bold;
        width: 100px;
        height: 100px;
        z-index: 1;
    }
</style>
