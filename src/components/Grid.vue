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
                            <input type="number" class="form-control" id="alivePercentage" step="0.05" min="0.00" max="1.0" v-model.number="alivePercentage" >
                        </div>
                    </div>

                    <div class="form-group row">
                        <label for="rowCount" class="col-sm-2 col-form-label px-0">Rows</label>
                        <div class="col-sm-7">
                            <input type="number" class="form-control" id="rowCount" step="1" v-model.number="rows">
                        </div>
                    </div>
                    <div class="form-group row">
                        <label for="columnCount" class="col-sm-2 col-form-label px-0">Columns</label>
                        <div class="col-sm-7">
                            <input type="number" class="form-control" id="columnCount" step="1" v-model.number="cols">
                        </div>
                    </div>
                    <div class="form-group row">
                        <div class="col-sm-10">
                            <button id="initialize" class="btn btn-primary" :disabled="!inputsAreValid" @click="initialize">Initialize</button>
                        </div>
                    </div>
                </div>
                <hr>
                <h3>Actions</h3>
                <div class="form-inline ">
                    <label class="form-check-label" for="stepCount">
                        Step: {{step}}
                    </label>
                    <button class="btn btn-primary ml-2" :disabled="!inputsAreValid" @click="nextStep">Next Step</button>
                </div>
            </div>
            <div class="col-md-6">
                <div class="row my-1" v-show="step === 0">
                    <div class="alert alert-primary" role="alert">
                        <h4 class="alert-heading">Hint:</h4>
                        <p>You can draw alive cells on left mouse click and dead cells on left mouse click. You can only do this before your first move.</p>
                    </div>
                </div>
                <div class="row mx-a" @mousedown.left="activateDraw(1)" @mousedown.right="activateDraw(0)" @mouseup="deactivateDraw" @mouseleave="deactivateDraw">
                    <div class="d-inline" v-for="(row, ridx) in grid" :key="ridx">
                        <div v-bind:class="[ grid[ridx][cidx] === 1 ? 'cell cell-alive' : 'cell cell-dead' ]" v-for="(value, cidx) in row" :key="cidx" 
                                @mouseover="draw(ridx, cidx, $event)" 
                                @click.left="pointDraw(ridx, cidx, 1, $event)"
                                @click.right="pointDraw(ridx, cidx, 0, $event)">
                        </div>
                    </div>
                </div>
                
            </div>
        </div>
  </div>
</template>

<script>
import axios from 'axios'

function initGrid(alivePercentage=0.25, rows=20, cols=20) {
    const grid = []

    for(let row = 0; row < rows; row++) {
        const newRow = [];
        for(let col = 0; col < cols; col++) {
            const val = Math.random() < alivePercentage ? 1 : 0;
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
          grid: g,
          alivePercentage: 0.25,
          rows: 20,
          cols: 20,
          drawIsActive: false,
          drawValue: 1
      }
  },

  methods: {
      nextStep() {

          const that = this;
          axios.post('http://localhost:8080/grid', {
              grid: this.grid 

          }).then((response) => {
              const newGrid = response.data.grid;
              this.step = this.step + 1;
              this.grid = [];

              that.updateGrid(newGrid);
          }).catch((err) => {
              alert('Something went wrong: ', err)
          });
      },

      initialize() {
            const g = initGrid(this.alivePercentage, this.rows, this.cols)
            this.step = 0;
            this.grid = g;
            this.drawIsActive = false;
            this.drawValue = 1;
      },

      updateGrid(newGrid) {
          this.grid = [];

          for (let i = 0; i < newGrid.length; i++) {
            let row = []
            for (let j = 0; j < newGrid[i].length; j++) {
                row.push(newGrid[i][j]);
            }
            this.grid.push(row);
          }
      },

      activateDraw(val) {
          this.drawIsActive = true;
          this.drawValue = val;
      },

      deactivateDraw() {
          this.drawIsActive = false;
      },

      draw: function(row, col, event) {
          if (this.step === 0 && this.drawIsActive) {
              const alteredGrid = [...this.grid];
              alteredGrid[row][col] = this.drawValue;

              this.updateGrid(alteredGrid);
          }
      },

    pointDraw: function(row, col, val, event) {
          if (this.step === 0) {
              const alteredGrid = [...this.grid];
              alteredGrid[row][col] = val;

              this.updateGrid(alteredGrid);
          }
      }
  },
  computed: {
      inputsAreValid() {
            if(this.alivePercentage === undefined 
                || this.alivePercentage < 0 
                || this.alivePercentage > 1) {

                return false;
            }

            if(this.rows === undefined 
                || this.rows < 1 
                || this.rows > 50) {

                return false;
            }

            if(this.cols === undefined 
                || this.cols < 1 
                || this.cols > 50) {

                return false;
            }

            return true;
      }
  }
}
</script>


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
