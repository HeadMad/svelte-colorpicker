<script>
  import Range from '$lib/range.svelte';
  let tx = $state(0);
  let ty = $state(0);
  let px = $state(0);
  let py = $state(0);
  function onmove(pos, prgrs) {
    px = prgrs.x;
    py = prgrs.y;
    tx = pos.left;
    ty = pos.top;
  }
</script>
<br>
px: [{px}] py: [{py}] tx: [{tx}] ty: [{ty}]
<br>

<Range {onmove} style="display: block">
  {#snippet thumb({top, left})}
    <div class="thumb" style="top: {top}px; left: {left}px"></div>
  {/snippet}
  {#snippet track()}
    <div class="track"></div>
  {/snippet}
</Range>

<style>
  .thumb {
    background-color: red;
    width: 20px;
    height: 20px;
    position: absolute;

  }

  .track {
    background-color: blue;
    /* width: 100px; */
    height: 20px;
  }
</style>