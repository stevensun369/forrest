<script>
  export let field
  export let fieldSize

  console.log("field size: " + $fieldSize)
</script>

<div id="filed">
  {#each [...Array($fieldSize).keys()] as i}
    <div class="line">
      {#each [...Array($fieldSize).keys()] as j}
        <div class="cell color-{$field[i][j]}"></div>
        <!-- <div class="cell color-0"></div> -->
      {/each}
    </div>
  {/each} 
</div>

<style scoped>
  .field {
    width: auto;
    height: auto;
    display: block;
  }
  .line {
    float: left;
    width: 100%;
    height: var(--cell);
  }

  .cell {
    height: var(--cell); 
    width: var(--cell);
    /* border: 0.5px solid black; */
    float: left;
    /* background: gray; */
  }

  .color-0 {
    background: #74663B;
  }

  .color-1 {
    background: #00ff11;
  }

  .color-2 {
    background: #162a00;
  }

  .color-3 {
    background: gray;
  }
</style>