<script>
   import Colorpicker from '$lib/colorpicker.svelte';

   let R = $state(0);
	let G = $state(0);
	let B = $state(0);
	let A = $state(0);

  let hex = $state("");

  function onchangecolor ({RGBA, HEX, HSLA}) {
    [R, G, B, A] = RGBA;
    hex = HEX;
  }
</script>

<main>
  R: [{R}] G: [{G}] B: [{B}] A: [{A}]
  <br>
  {hex}
  <Colorpicker {onchangecolor} size={240}/>
</main>

<style>

</style>