<template>
  <div id="grid" class="mt-4">
      
        <div class="row">
            <div class="col-md-4 border-right mr-2">
                <h3>Settings</h3>
                <hr>
                <div>
                    <div class="form-group row">
                        <label for="alivePercentage" class="col-sm-2 col-form-label px-0">Alive %</label>
                        <div class="col-sm-7">
                            <input type="number" class="form-control" id="alivePercentage" step="0.05" min="0.05" max="1.0" v-model="alivePercentage" >
                        </div>
                    </div>

                    <div class="form-group row">
                        <label for="rowCount" class="col-sm-2 col-form-label px-0">Rows</label>
                        <div class="col-sm-7">
                            <input type="number" class="form-control" id="rowCount" step="1" v-model="rows">
                        </div>
                    </div>
                    <div class="form-group row">
                        <label for="columnCount" class="col-sm-2 col-form-label px-0">Columns</label>
                        <div class="col-sm-7">
                            <input type="number" class="form-control" id="columnCount" step="1" v-model="cols">
                        </div>
                    </div>
                    <div class="form-group row">
                        <div class="col-sm-10">
                            <button id="initialize" class="btn btn-primary" @click="initialize">Initialize</button>
                        </div>
                    </div>
                </div>
                <hr>
                <h3>Actions</h3>
                <div class="form-inline ">
                    <label class="form-check-label" for="stepCount">
                        Step: {{step}}
                    </label>
                    <button class="btn btn-primary ml-2" @click="nextStep">Next Step</button>
                </div>
            </div>
            <div class="col-md-6">
                <div class="row mx-a">
                    <div class="d-inline" v-for="(row, ridx) in grid" :key="ridx">
                        <div v-bind:class="[ grid[ridx][cidx] === 1 ? 'cell cell-alive' : 'cell cell-dead' ]" v-for="(value, cidx) in row" :key="cidx">
                        </div>
                    </div>
                </div>
                
            </div>
        </div>
  </div>
</template>

<script>
import axios from 'axios'

function initGrid(alive="0.25", r=20, c=20) {
    const grid = []
    const alivePercentage = parseFloat(alive)
    const rows = parseInt(r,10);
    const cols = parseInt(c,10);

    for(let row = 0; row < rows; row++) {
        const newRow = [];
        for(let col = 0; col < cols; col++) {
            const val = Math.random() < alive ? 1 : 0;
            newRow.push(val);
        }
        grid.push(newRow);   
    }

    return grid;
}

export default {
  name: 'Grid',
  props: {
  },
  data() {
      const g = initGrid()
      return {
          step: 0,
          history: [g],
          grid: g,
          alivePercentage: 0.75,
          rows: 20,
          cols: 20
      }
  },

  methods: {
      nextStep() {
          axios.post('http://localhost:8080/grid', {
              grid: this.grid 

          }).then((response) => {
              const newGrid = response.data.grid;
              this.history.push(newGrid);
              this.step = this.step + 1;
              this.grid = [];

              for (let i = 0; i < newGrid.length; i++) {
                  let row = []
                  for (let j = 0; j < newGrid[i].length; j++) {
                      row.push(newGrid[i][j]);
                  }
                  this.grid.push(row);
              }
          });
      },
      initialize() {
            const g = initGrid(this.alivePercentage, this.rows, this.cols)
            this.step = 0;
            this.history = [g];
            this.grid = g;
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

    .cell {
        width: 20px;
        height: 20px;
        border: 1px solid silver;
    }

    .cell-dead {
        background-color: white;
    }

    .cell-alive {
        background-color: green;
    }
</style>
