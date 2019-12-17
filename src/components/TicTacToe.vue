<template>
    <div class="hello">
        <h1>{{ msg }}</h1>
        <div class="field">
            <div class="cell" :id="i" v-for="(cell, i) in cells" :key="i" @click="onCellClicked(i)">
                <div class="symbol" :class="cell"></div>
            </div>
        </div>
        <p v-if="winner === 'AI'">Game over, {{winner}} won.</p>
        <p v-else-if="winner === 'Player'">You win! Cheers.</p>
        <p v-else-if="winner === 'nobody'">We have a tie.</p>
    </div>
</template>

<script>
    export default {
        name: 'TicTacToe',
        props: {
            msg: String
        },
        data: function () {
            return {
                cells: (Array(9)).fill(null),
                playerSymbol: 'x',
                AISymbol: 'o',
                winningCombinations: [
                    [0, 1, 2],
                    [3, 4, 5],
                    [6, 7, 8],
                    [0, 3, 6],
                    [1, 4, 7],
                    [2, 5, 8],
                    [0, 4, 8],
                    [2, 4, 6]
                ],
                winner: null,

            }
        },
        watch: {
            cells: function () {
                if (this.isWin(this.AISymbol, this.cells)) {
                    this.winner = 'AI'
                } else if (this.isWin(this.playerSymbol, this.cells)) {
                    this.winner = 'Player'
                } else if (this.emptyCells(this.cells).length === 0) {
                    this.winner = 'nobody'
                }
            }
        },
        methods: {
            emptyCells(field) {
                return Object.keys(field).filter(item => field[item] === null)
            },
            onCellClicked(cellId) {
                if (this.cells[cellId] !== null) {
                    return;
                }
                if (this.winner) {
                    return;
                }
                this.cells[cellId] = this.playerSymbol
                const nextMove = this.calcMinmaxScore(Object.assign({}, this.cells), this.AISymbol)

                this.$set(this.cells, nextMove.cellId || this.emptyCells(this.cells)[0], this.AISymbol)
            },
            isWin(symbol, cells) {
                return this.winningCombinations.some(seq => {
                    return cells[seq[0]] === symbol && cells[seq[1]] === symbol && cells[seq[2]] === symbol;
                })
            },
            calcMinmaxScore(field, symbol) {
                const emptyCells = this.emptyCells(field);

                if (this.isWin(this.playerSymbol, field)) {
                    return {score: -1 * (emptyCells.length + 1)}
                } else if (this.isWin(this.AISymbol, field)) {
                    return {score: emptyCells.length + 1}
                } else if (emptyCells.length === 0) {
                    return {score: 0};
                }
                let moves = []

                // nobody win yet
                emptyCells.forEach((emptyCell) => {
                    let move = {}
                    move.cellId = emptyCell
                    field[emptyCell] = symbol
                    move['score'] = (this.calcMinmaxScore(field, (symbol === this.AISymbol ? this.playerSymbol : this.AISymbol))).score
                    field[emptyCell] = null //clear fake moving
                    moves.push(move);
                })
                if (symbol === this.AISymbol) {
                    moves = moves.sort((a, b) => b.score - a.score)
                } else {
                    moves = moves.sort((a, b) => a.score - b.score)
                }

                return moves[0];
            }
        }
    }
</script>

<style scoped>
    h3 {
        margin: 40px 0 0;
    }

    .field {
        display: flex;
        flex-wrap: wrap;
        width: 150px;
        margin: auto;
    }

    .cell {
        width: 50px;
        height: 50px;
        border: 1px solid #999;
        box-sizing: border-box;
    }


    .symbol {
        width: 100%;
        height: 100%;
        background-size: 75%;
        background-repeat: no-repeat;
        background-position: center;
        opacity: 0;
        transition: opacity .15s ease-in;
    }

    .symbol.x {
        background-image: url("../assets/x.svg");
        opacity: 1;
    }

    .symbol.o {
        background-image: url("../assets/o.svg");
        opacity: 1;
    }
</style>
