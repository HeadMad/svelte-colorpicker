<script>
  let {thumb, track, style = "", onmove, onstart, onend} = $props();

  const isTouch =
		"ontouchstart" in globalThis ||
		(globalThis.DocumentTouch &&
			document instanceof globalThis.DocumentTouch) ||
		navigator.maxTouchPoints > 0 ||
		globalThis.navigator.msMaxTouchPoints > 0;

  let range = $state();
  let isPushed = $state(false);

  let w;
  let h;

  let position = $state({left: 0, top: 0});
  let progress = $derived({
    get x() {
      return Number(position.left / w).toFixed(2);
    },
    get y() {
      return Number(position.top / h).toFixed(2);
    },
  });

  let box;

  $effect(() => {
    w = range.offsetWidth;
    h = range.offsetHeight;

    box = range.getBoundingClientRect();

    range.addEventListener('mousedown', start);
    document.addEventListener('mouseup', end);
    document.addEventListener('mousemove', move);

    return () => {
      range.removeEventListener('mousedown', start);
      document.removeEventListener('mouseup', end);
      document.removeEventListener('mousemove', move);
    };
  });

  function start(e) {
    isPushed = true;
    position = getPosition(e);
    onstart(position, progress);
  }

  function end(e) {
    isPushed = false;
    onend();
  }

  function move(e) {
    if (!isPushed) return;
    position = getPosition(e);
    onmove(position, progress);
  }

  /**
	 *
	 * @param e
	 */
	function getPosition(e) {
    let left = null, top = null;
		return {
      get left() {
        if ( left !== null ) return left;
				const pageX = isTouch ? e.touches[0].pageX : e.pageX;
				const value = Math.ceil(pageX - box.left - window.pageXOffset);
				return left = clamp(0, w, value);
      
			},
			get top() {
        if ( top !== null ) return top;
				const pageY = isTouch ? e.touches[0].pageY : e.pageY;
				const value = Math.ceil(pageY - box.top - window.pageYOffset);
				return top = clamp(0, h, value);
			}
		};
	}


  /**
	 *
	 * @param min
	 * @param max
	 * @param curr
	 */
	function clamp(min, max, curr) {
		return Math.max(Math.min(curr, max), min);
	}

</script>

<div
class="range"
bind:this={range}
{style}
>
  {@render track(position, progress)}
  {@render thumb(position, progress)}
</div>

<style>
  .range {
    position: relative;
    display: inline-block;
  }
</style>