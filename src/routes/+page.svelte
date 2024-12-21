<script>
   import Colorpicker from '$lib/colorpicker.svelte';

   let R = $state(0);
	let G = $state(0);
	let B = $state(0);

  function onchangecolor ({RGBA, HEX, HSLA}) {
    [R, G, B] = RGB;
  }
</script>

<main>
  R: [{R}] G: [{G}] B: [{B}]
  <Colorpicker />
</main>

<style>

</style>