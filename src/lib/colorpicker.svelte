<script>

	let hR = $state(255);
	let hG = $state(0);
	let hB = $state(0);
	
	let cx = $state(0);
	let cy = $state(0);
	let hy = $state(0);

	let color = $state();
	let hue = $state();
	
	let R = $state(0);
	let G = $state(0);
	let B = $state(0);

	let tmp = $state(0);

	let isColorMouseDown = $state(false);
	let isHueMouseDown = $state(false);


	const hueSections = [
		value => changeHue(255, value, 0),
		value => changeHue(255 - value, 255, 0),
		value => changeHue(0, 255, value),
		value => changeHue(0, 255 - value, 255),
		value => changeHue(value, 0, 255),
		value => changeHue(255, 0, 249 - value),
	];

	function changeHue(r, g, b) {
		hR = r;
		hG = g;
		hB = b;
		setRGB();
	}

	$effect(() => {
		if (isColorMouseDown || isHueMouseDown)
			document.body.classList.add('unselectable');
		else
			document.body.classList.remove('unselectable');
	});
	
	let hueSectionHeight = 0;
	
	$effect(() => {
		hueSectionHeight = hue?.offsetHeight / 6;
		
		document.addEventListener('mousedown', (e) => {
			if (e.target == color) {
				isColorMouseDown = true;
				colorMouseMove(e);
			}

			if (e.target == hue) {
				isHueMouseDown = true;
				hueMouseMove(e);
			}
			
		});

		document.addEventListener('mouseup', (e) => {
			isColorMouseDown = false;
			isHueMouseDown = false;
		});

		document.addEventListener('mousemove', (e) => {
			if (isColorMouseDown)
				colorMouseMove(e);

			
			if (isHueMouseDown)
				hueMouseMove(e);
		});
	});

	function colorMouseMove(e) {
		const coord = getCoordinates(e, color);
		cx = coord.x;
		cy = coord.y;

		setRGB();
	}

	function setRGB() {
		const h = color.offsetHeight;
		const w = color.offsetWidth;
		const setColor = (hRGB) => {
			// return hRGB + valueInRange(255 - hRGB, 255, 255 - cx);
			const xRGB = hRGB + valueInRange(255 - hRGB, w, w - cx);
			return  Math.round(valueInRange(xRGB * 255, h, h - cy)/255);
		};
		R = setColor(hR);
		G = setColor(hG);
		B = setColor(hB);
	}

	function hueMouseMove(e) {
		const coord = getCoordinates(e, hue);
		hy = coord.y;
		const s = Math.floor(hy/hueSectionHeight);
		const rem = hy%hueSectionHeight;

		const val = valueInRange(255, hueSectionHeight, rem);
		hueSections[s](val);
	}

	function getCoordinates(e, target) {
		const box = target.getBoundingClientRect();
		let x, y;
		return {
		 get y() {
		   y = Math.ceil(e.pageY - box.top + window.pageYOffset);
			 return clamp(0, target.offsetHeight, y);
		 },
		 get x() {
		   x = Math.ceil(e.pageX - box.left + window.pageXOffset);
			 return clamp(0, target.offsetWidth, x);
		 }
		};
	}

function clamp(min, max, curr) {
	return Math.max(Math.min(curr, max), min);
}

function valueInRange(newMax, curMax, curValue) {
	return Math.round(curValue * newMax /curMax);
}



</script>
<div class="rgb">
<span class="square" style="background: rgb({R}, {G}, {B})"></span>
<span></span>
</div>
R: [{R}] G: [{G}] B: [{B}]
<br>
hR: [{hR}] hG: [{hG}] hB: [{hB}]
<br>
cy: {cy} cx: {cx}

<div class="body">
	<div
		class="color"
		style="background-color: rgb({hR}, {hG}, {hB})"
		id="color"
		class:nocursor={isColorMouseDown}
		bind:this={color}
		>
		<div class="color-picker" style="
			left:{cx}px;
			top:{cy}px;
			"></div>
		</div>
	<div
		class="hue"
		class:nocursor={isHueMouseDown}
		bind:this={hue}
		>
		<div class="hue-picker" style="top:{hy}px"></div>
		</div>
</div>

<style>

	.rgb {
		height: 2rem;
		line-height: 2rem;
		margin: 1rem;
		align-content: baseline;
		/* background: #ccc; */
	}

	.square {
		height: 2rem;
		width: 2rem;
		margin-right: 1rem;
		display: inline-block;
		float:left;
	}

	.body {
		gap: 1rem;
		display: flex;
		width: max-content;
		border: 1px solid #ccc;
		padding: 1rem;
	}
	.hue {
		position: relative;
		width: 40px;
		height: 240px;
		background-image: linear-gradient(0deg, red 0, #f0f 17%, #00f 33%, #0ff 50%, #0f0 67%, #ff0 83%, red);
		/* background-image: linear-gradient(0deg, red 0, #ff0 17%, #0f0 33%, #0ff 50%, #00f 67%, #f0f 83%, red); */
	}
	.color {
		width: 240px;
		/* height: 200px; */
		position: relative;
		background-color: #ff0000;
		background-image: linear-gradient(0deg, #000, rgba(204, 154, 129, 0)),
			linear-gradient(90deg, #fff, rgba(204, 154, 129, 0));
	}

	.nocursor {
		cursor: none;
	}


	.color-picker {
		position: absolute;
		width: 8px;
		height: 8px;
		border: 2px solid #fff;
		box-shadow: 0 0 3px rgba(0, 0, 0, .3);
		border-radius: 50%;
		transform: translate(-50%, -50%);
	}


	.hue-picker {
		position: absolute;
		width: 100%;
		height: 1px;
		padding: 0 2px;
		left: -2px;
		box-shadow: 0 0 3px rgba(0, 0, 0, .3);
		border: 1px solid #fff;
		transform: translateY(-50%);
		opacity: 1;
	}

	

		:global(.unselectable) {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
</style>