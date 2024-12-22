<script>
	let { onchangecolor, startcolor, size = 240 } = $props();

	const isTouch =
		"ontouchstart" in globalThis ||
		(globalThis.DocumentTouch &&
			document instanceof globalThis.DocumentTouch) ||
		navigator.maxTouchPoints > 0 ||
		globalThis.navigator.msMaxTouchPoints > 0;

	let hR = $state(255);
	let hG = $state(0);
	let hB = $state(0);

	let cx = $state(size);
	let cy = $state(0);
	let hy = $state(0);
	let ax = $state(size);

	let color = $state();
	let hue = $state();
	let alpha = $state();

	let R = $state(255);
	let G = $state(0);
	let B = $state(0);
	let A = $state(1);

	/**
	 * Object is selected
	 * @type {boolean|color|hue|alpha}
	 */
	let selected = $state(false);

	const hueSectionHandlers = [
		(value) => setHue(255, value, 0),
		(value) => setHue(255 - value, 255, 0),
		(value) => setHue(0, 255, value),
		(value) => setHue(0, 255 - value, 255),
		(value) => setHue(value, 0, 255),
		(value) => setHue(255, 0, 255 - (value || 255)),
	];

	const mouseDownHandlers = new Map();

	const eventHandlers = [
		(e) => {
			selected = mouseDownHandlers.has(e.target)
				? e.target
				: mouseDownHandlers.has(e.target.parentNode)
					? e.target.parentNode
					: false;
			selected && mouseDownHandlers.get(selected)(e);
		},
		(e) => selected && mouseDownHandlers.get(selected)(e),
		(e) => (selected = false),
	];

	const eventNames = new Map();
	eventNames.set(true, ["touchstart", "touchmove", "touchend"]);
	eventNames.set(false, ["mousedown", "mousemove", "mouseup"]);

	$effect(() => {
		runChangeHandler();

		mouseDownHandlers.set(color, colorMouseMove);
		mouseDownHandlers.set(hue, hueMouseMove);
		mouseDownHandlers.set(alpha, alphaMouseMove);

		eventNames.get(isTouch).forEach((name, i) => {
			document.addEventListener(name, eventHandlers[i]);
		});

		return () => {
			eventNames.get(isTouch).forEach((name, i) => {
				document.removeEventListener(name, eventHandlers[i]);
			});
		};
	});

	$effect(() => {
		if (selected) document.body.classList.add("unselectable");
		else document.body.classList.remove("unselectable");
	});

	/**
	 *
	 */
	function setRGB() {
		const setColor = (hRGB) => {
			const xRGB = hRGB + valueInRange(255 - hRGB, size, size - cx);
			return Math.round(valueInRange(xRGB * 255, size, size - cy) / 255);
		};

		R = setColor(hR);
		G = setColor(hG);
		B = setColor(hB);

		runChangeHandler();
	}

	/**
	 *
	 * @param r
	 * @param g
	 * @param b
	 */
	function setHue(r, g, b) {
		hR = r;
		hG = g;
		hB = b;
		setRGB();
	}

	/**
	 *
	 * @param e
	 */
	function colorMouseMove(e) {
		const coord = getCoordinates(e, color);
		cx = coord.x;
		cy = coord.y;

		setRGB();
	}

	/**
	 *
	 * @param e
	 */
	function hueMouseMove(e) {
		const coord = getCoordinates(e, hue);
		hy = coord.y;
		const height = size / 6;
		const s = clamp(0, 5, Math.floor(hy / height));
		const rem = hy % height;

		const val = valueInRange(255, height, rem);
		hueSectionHandlers[s](val);
	}

	/**
	 *
	 * @param e
	 */
	function alphaMouseMove(e) {
		const coord = getCoordinates(e, alpha);
		ax = coord.x;

		A = Number(ax / size).toFixed(2);

		runChangeHandler();
	}

	/**
	 *
	 */
	function runChangeHandler() {
		onchangecolor({
			RGBA: [R, G, B, A],
			get HEX() {
				return `#${R.toString(16).padStart(2, "0")}${G.toString(16).padStart(2, "0")}${B.toString(16).padStart(2, "0")}${Math.round(
					A * 255,
				)
					.toString(16)
					.padStart(2, "0")}`;
			},
			HSLA: [],
		});
	}

	/**
	 *
	 * @param e
	 * @param target
	 */
	function getCoordinates(e, target) {
		const box = target.getBoundingClientRect();
		let x, y;
		return {
			get y() {
				const pageY = isTouch ? e.touches[0].pageY : e.pageY;
				y = Math.ceil(pageY - box.top - window.pageYOffset);
				return clamp(0, size, y);
			},
			get x() {
				const pageX = isTouch ? e.touches[0].pageX : e.pageX;
				x = Math.ceil(pageX - box.left - window.pageXOffset);
				return clamp(0, size, x);
			},
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

	/**
	 *
	 * @param newMax
	 * @param curMax
	 * @param curValue
	 */
	function valueInRange(newMax, curMax, curValue) {
		return Math.round((curValue * newMax) / curMax);
	}
</script>
<div
	class="body"
	style="
	--RGB: {R}, {G}, {B};
	--A: {A};
	--hue: {hR}, {hG}, {hB};
	--size: {size}px;
	--width: {size / 10}px;"
>
	<div
		class="color"
		style="background-color: rgb(var(--hue));"
		id="color"
		class:nocursor={selected === color}
		bind:this={color}
	>
		<div
			class="picker"
			style="
			background-color: rgb( var(--RGB));
			left:{cx}px;
			top:{cy}px;
			"
		></div>
	</div>

	

	<div class="hue line" class:nocursor={selected === hue} bind:this={hue}>
		<div
			class="picker"
			style="
		background-color: rgb(var(--hue));
		top:{hy}px;
		left: 50%;
		"
		></div>
	</div>
	<div class="alpha line" class:nocursor={selected === alpha} bind:this={alpha}>
		<div
			class="picker"
			style="
		background-color: rgba(var(--RGB), var(--A));
		left:{ax}px;
		top: 50%;
		"
		></div>
	</div>
	<div class="square" style="background-color: rgba(var(--RGB), var(--A));"></div>
</div>

<style>


	.square {
		height: var(--width);
		width: var(--width);
		box-sizing: border-box;
		border: 2px solid #fff;
		border-radius: .3rem;
		box-shadow: 0 0 5px rgba(0, 0, 0, 0.15);
	}

	.body {
		display: grid;
		grid-template: auto auto / auto auto;
		gap: .7rem;
		width: max-content;
		border: 1px solid #ccc;
		padding: .7rem;
	}


	.hue {
		position: relative;
		width: var(--width);
		height: var(--size);
		background-image: linear-gradient(
			0deg,
			red 0,
			#f0f 16.6%,
			#00f 33.2%,
			#0ff 49.8%,
			#0f0 66.4%,
			#ff0 83%,
			red
		);
	}
	.color {
		width: var(--size);
		height: var(--size);
		position: relative;
		background-color: #ff0000;
		background-image: linear-gradient(0deg, #000, rgba(204, 154, 129, 0)),
			linear-gradient(90deg, #fff, rgba(255, 255, 255, 0));
	}

	.alpha {
		position: relative;
		height: var(--width);
		width: var(--size);
		background-image: linear-gradient(90deg, #ffffff00, rgb(var(--RGB))),
			url("data:image/svg+xml,%3Csvg viewBox='0 0 2 2' xmlns='http://www.w3.org/2000/svg'%3E%3Crect width='1' height='1' fill='%230000001a'/%3E%3Crect x='1' y='1' width='1' height='1' fill='%230000001a'/%3E%3C/svg%3E");
		background-size: auto 10px;
	}

	.nocursor {
		cursor: none;
	}

	.picker {
		position: absolute;
		width: 10px;
		height: 10px;
		border: 2px solid #fff;
		box-shadow: 0 0 3px rgba(0, 0, 0, 0.6);
		border-radius: 50%;
		transform: translate(-50%, -50%);
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
