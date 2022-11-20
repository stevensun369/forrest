<script>
  import { onMount } from "svelte"
  import { get, writable } from "svelte/store"
  import Field from "./Field.svelte"

  let fieldSize = writable(100)

  let initU = 3
  let initV = 3
  let initW = 5

  let u = [initU]
  let v = [initV]
  let w = [initW]

  let sEst = [0]
  let yMort = [0]
  let mat = [0]
  let oMort = [0]
  let sDiff = [0]
  let sDep = [0]
  let sProd = [0]

  let gen = 0

  let initGens = writable(100)
  let generations = writable([...Array($initGens).keys()])

  let alfa = writable(5) // seed production rate
  let d = writable(2) // seed diffusivity rate
  let beta = writable(1) // seed deposition rate
  let delta = writable(0.4) // seed establishement rate
  let gamma = writable(0.02) // young tree death rate
  let h = writable(0.02) // old tree death rate
  let f = writable(0.4) // maturization rate

  let field = writable([])

  onMount(() => {
    for (let i in [...Array(100).keys()]) {
      $field.push([])
      for (let j in [...Array(100).keys()]) {
        $field[i].push(0)
      }
    }
  })

  function modulus(n) {
    if (n < 0) {
      return -n
    } else {
      return n
    } 
  }

  function calcU() {
    sEst[gen + 1] = $beta * modulus(w[gen]) * $delta
    yMort[gen + 1] = $gamma * u[gen]
    mat[gen + 1] = $f * u[gen]

    u[gen + 1] = sEst[gen + 1] - yMort[gen + 1] - mat[gen + 1]


    return ''
  }

  function calcV() {
    mat[gen + 1] = $f * u[gen]
    oMort[gen + 1] = $h * v[gen]

    v[gen + 1] = mat[gen + 1] - oMort[gen + 1 ]

    return ''
  }

  function calcW() { 
    sDiff[gen + 1] = $d * w [gen]
    sDep[gen + 1] = $beta * w[gen]
    sProd[gen + 1] = $alfa * modulus(v[gen])
    w[gen + 1] = sDiff[gen + 1] - sDep[gen + 1] + sProd[gen + 1]
    //         s. diffusivity  s. deposition   s.production

    return ''
  }

  function incrGen() {
    gen = gen + 1;

    return ''
  }

  function nextGen() {
    calcU();
    calcV();
    calcW();
    incrGen();
    return ''
  }

  function restart() {
    $generations = [...Array($initGens).keys()]
    for (let i in $generations) {
      u[i] = 0;
      v[i] = 0;
      w[i] = 0;
    }
    u[0] = initU;
    v[0] = initV;
    w[0] = initW;
    gen = 0;
  }

  $: {
    for (let i in $generations) {
      nextGen()
    }

    let cU = Math.round(u[gen]);
    let cV = Math.round(v[gen]);

    console.log('--- before ---')
    console.log("copy of U: " + cU)
    console.log("copy of V: " + cV)

    for (let i in [...Array($fieldSize).keys()]) {
      $field.push([])
      for (let j in [...Array($fieldSize).keys()]) {
        $field[i][j] = 0
      }
    }

    let c = 0;
    while (c < ($fieldSize * $fieldSize)) {
      c++
      let i = Math.floor(Math.random() * $fieldSize)
      let j = Math.floor(Math.random() * $fieldSize)

      let decision = Math.floor(Math.random() * 3)

      if (cV <= 0 && decision === 2) {
        decision = 1
      }

      if (cU <= 0 && decision === 1) {
        decision = 0
      }

      if (decision === 1) {
        cU -= 1
      }

      if (decision === 2) {
        cV -= 1
      }

      if ($field[i][j] === 0) {
        let cField = get(field)

        cField[i][j] = decision

        field.set(cField)
      }
    }
    console.log('--- after ---')
    console.log("copy of U: " + cU)
    console.log("copy of V: " + cV)
  }
</script>

<main>
  
  <div id="inputs">
    <div style="height: 10px;"></div>
    u inițial: <input  type="text" bind:value={initU}>
    v inițial: <input  type="text" bind:value={initV}>
    w inițial: <input  type="text" bind:value={initW}> 
    <br>
    α (rata producție semințe): <input type="text" bind:value={$alfa}> <br>
    d (rata difusivitate semințe): <input type="text" bind:value={$d}> <br>
    β (rata depunere semințe): <input type="text" bind:value={$beta}> <br>
    δ (rata stabilire semințe): <input type="text" bind:value={$delta}> <br>
    γ (rata mortalității tinere): <input type="text" bind:value={$gamma}> <br>
    h (rata mortalității bătrâne): <input type="text" bind:value={$h}> <br>
    f (rata maturizării): <input type="text" bind:value={$f}> <br>

    numar generatii: <input type="text" bind:value={$initGens}> <br>

  </div>

  <input id="restart" type="submit" value="restart" on:click={restart}><br>

  
  <Field {fieldSize} {field} />
  
  <!-- {#each $generations as i}
    {i}: <br>
    --- (u = {u[i]}) - (sEst = {sEst[i]}) - (yMort = {yMort[i]}) - (mat = {mat[i]})  <br>
    --- (v = {v[i]}) - (mat = {mat[i]}) - (oMort = {oMort[i]}) <br>
    --- (w = {w[i]}) - (sDiff = {sDiff[i]}) - (sDep = {sDep[i]}) - (sProd = {sProd[i]}) <br>
  {/each} -->
  {#each $generations as i}
    <div class="heading">{i}</div>
    (u = {u[i]}) <br>
    (v = {v[i]}) <br>
    (w = {w[i]}) <br>
    <br>
  {/each}
</main>

<style>
  :root {
    --cell: 5px;
    --lightgreen: #38E377;
    --darkgreen: #054D1B;
    --brown: rgb(116,102,59);
  }

  #inputs {
    font-family: monospace;
  }

  .heading {
    font-weight: 600;
  }

  #inputs input {
    width: 50px;
    height: 30px;
    border-radius: 300px;
    padding-left: 10px;
    outline: none;
    border: none;
    border: 1px solid var(--darkgreen);
    margin-bottom: 5px;
  }

  #restart {
    width: 100px;
    height: 30px;
    background: #00ff11;
    outline: none;
    color: white;
    border: none;
    border-radius:300px;

    margin-top: 10px;
    margin-bottom: 10px;
  }
</style>

