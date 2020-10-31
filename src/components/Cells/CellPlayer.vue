<template>
    <div>

    </div>
</template>

<script>
    export default {
        props: ["hasX", "hasY"],
        computed: {
            getNumber : function() {
                return Number(Number(this.hasX) * 10) + Number(this.hasY)
            }
        },
        methods: {
            move: function(targetComp, dir, executeCallback) {
                return new Promise((resolve, reject) => {
                    let queryString =
                        `PREFIX : <http://snowman/antoine/erwann#>
                         PREFIX swrl: <http://www.semanticweb.org/21504712/ontologies/2020/9/jouer#>

                         DELETE {
                             ?target a :CellFree .
                             ?player a :CellPlayer .
                         }
                         INSERT {
                             ?target a :CellPlayer .
                             ?player a :CellFree .
                         }
                         WHERE {
                             ?player :${dir} ?target ;
                                    a :CellPlayer .

                             ?player swrl:movableOn ?target .
                         }`

                    executeCallback(queryString, true)
                    .then(() => {
                        resolve()
                    }).catch(e => reject(e))
                })
              console.log("PlayerMOOVE");

            }
        }
    }
</script>

<style scoped>
    div {
        background-size: cover;
        background-image: url("../../assets/witch.jpg");
        text-align: center;
        font-weight: bold;
        width: 100%;
        height: 100%;
    }
</style>
