<template>
    <div class="cell">

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
            move: function() {
                return new Promise((resolve) => {
                    resolve()
                })
            },
            isAllowedMove: function () {
                return new Promise((resolve) => {
                    resolve(true);
                });
            }
        }
    }
</script>

<style scoped>
    .cell {
        background-image: url("../../assets/dirt.jpg");
        text-align: center;
        font-weight: bold;
        width: 60px;
        height: 60px;
        z-index: 1;
    }
</style>
