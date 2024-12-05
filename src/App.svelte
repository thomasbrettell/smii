<script lang="ts" context="module">
  const SVG_SIZE = 50;
  const MIN_X_DISTANCE = 32;
  const MIN_X_DISTANCE_FROWN = 12;
</script>

<script lang="ts">
  import * as d3 from "d3";
  import { writable, type Writable } from "svelte/store";
  import Anchor from "./lib/Anchor.svelte";
  import Header from "./lib/Header.svelte";
  import { BG_COLOR, SMILE_COLOR } from "./lib/constants";
  import { onMount, tick } from "svelte";
  import { randomRange } from "./lib/utils";

  const curveLineGen = d3
    .line()
    .x((d) => d[0])
    .y((d) => d[1])
    .curve(d3.curveBasis);

  const lineGen = d3
    .line()
    .x((d) => d[0])
    .y((d) => d[1]);

  let windowWidth = window.innerWidth;
  let windowHeight = window.innerHeight;

  $: scaleRatio = Math.min(windowWidth, windowHeight) / 50;

  $: s = (val: number) => val / scaleRatio;

  let svgRef: SVGElement;
  let faceGroupRef: SVGGElement;

  let hideControls = false;
  let mouseDown = false;

  const mouthPoint1 = writable([6.866, 23.666]);
  const mouthPoint2 = writable([9.117, 41.293]);
  const mouthPoint3 = writable([41.918, 40.537]);
  const mouthPoint4 = writable([43.134, 23.552]);
  const eye1Point = writable(
    $mouthPoint1[1] <= $mouthPoint2[1]
      ? [$mouthPoint1[0] + 10, $mouthPoint1[1] + -10]
      : [$mouthPoint2[0], $mouthPoint2[1] - 10]
  );
  const eye2Point = writable(
    $mouthPoint4[1] <= $mouthPoint3[1]
      ? [$mouthPoint4[0] + -10, $mouthPoint4[1] - 10]
      : [$mouthPoint3[0], $mouthPoint3[1] - 10]
  );

  $: {
    eye1Point.set(
      $mouthPoint1[1] <= $mouthPoint2[1]
        ? [$mouthPoint1[0] + 10, $mouthPoint1[1] + -10]
        : [$mouthPoint2[0], $mouthPoint2[1] - 10]
    );
  }

  $: {
    eye2Point.set(
      $mouthPoint4[1] <= $mouthPoint3[1]
        ? [$mouthPoint4[0] + -10, $mouthPoint4[1] - 10]
        : [$mouthPoint3[0], $mouthPoint3[1] - 10]
    );
  }

  let controlledPointRef: null | Writable<number[]> = null;

  $: mouthPoints = [$mouthPoint1, $mouthPoint2, $mouthPoint3, $mouthPoint4] as [
    number,
    number,
  ][];
  $: curveLine = curveLineGen(mouthPoints);
  $: line = lineGen(mouthPoints);

  let bbox: SVGRect | undefined;
  $: if (faceGroupRef) {
    bbox = faceGroupRef.getBBox();
    $mouthPoint1;
    $mouthPoint2;
    $mouthPoint3;
    $mouthPoint4;
  }

  const randomise = async () => {
    mouthPoint1.set([
      randomRange(SVG_SIZE * 0.25, SVG_SIZE * 0.5),
      randomRange(SVG_SIZE * 0.25, SVG_SIZE),
    ]);
    mouthPoint2.set([
      randomRange($mouthPoint1[0], SVG_SIZE * 0.5),
      randomRange(SVG_SIZE * 0.25, SVG_SIZE),
    ]);

    const isFrown = $mouthPoint1[1] >= $mouthPoint2[1];

    mouthPoint3.set([
      randomRange(SVG_SIZE * 0.5, SVG_SIZE),
      randomRange(SVG_SIZE * 0.25, SVG_SIZE),
    ]);

    mouthPoint4.set([
      randomRange(
        Math.max($mouthPoint3[0], $mouthPoint4[0] - $mouthPoint1[0]),
        SVG_SIZE
      ),
      randomRange(
        isFrown ? $mouthPoint3[1] : SVG_SIZE * 0.25,
        isFrown ? SVG_SIZE : $mouthPoint3[1]
      ),
    ]);

    const difference = isFrown
      ? Math.max(MIN_X_DISTANCE_FROWN - ($mouthPoint3[0] - $mouthPoint2[0]), 0)
      : Math.max(MIN_X_DISTANCE - ($mouthPoint4[0] - $mouthPoint1[0]), 0);

    if (isFrown) {
      mouthPoint3.set([$mouthPoint3[0] + difference, $mouthPoint3[1]]);
    } else {
      mouthPoint4.set([$mouthPoint4[0] + difference, $mouthPoint4[1]]);
    }

    eye1Point.set(
      $mouthPoint1[1] <= $mouthPoint2[1]
        ? [$mouthPoint1[0] + 10, $mouthPoint1[1] + -10]
        : [$mouthPoint2[0], $mouthPoint2[1] - 10]
    );

    eye2Point.set(
      $mouthPoint4[1] <= $mouthPoint3[1]
        ? [$mouthPoint4[0] + -10, $mouthPoint4[1] - 10]
        : [$mouthPoint3[0], $mouthPoint3[1] - 10]
    );

    await tick();

    center();
  };

  const center = async () => {
    bbox = faceGroupRef.getBBox();

    const xTranslate = (SVG_SIZE - bbox.width) / 2 - bbox.x;
    const yTranslate = (SVG_SIZE - bbox.height) / 2 - bbox.y;

    mouthPoint1.set([
      $mouthPoint1[0] + xTranslate,
      $mouthPoint1[1] + yTranslate,
    ]);
    mouthPoint2.set([
      $mouthPoint2[0] + xTranslate,
      $mouthPoint2[1] + yTranslate,
    ]);
    mouthPoint3.set([
      $mouthPoint3[0] + xTranslate,
      $mouthPoint3[1] + yTranslate,
    ]);
    mouthPoint4.set([
      $mouthPoint4[0] + xTranslate,
      $mouthPoint4[1] + yTranslate,
    ]);

    await tick();
    bbox = faceGroupRef.getBBox();
  };

  onMount(async () => {
    document.documentElement.style.setProperty("--bg-color", BG_COLOR);
    document.documentElement.style.setProperty("--smile-color", SMILE_COLOR);
  });
</script>

<svelte:window
  on:mousemove={(e) => {
    if (!mouseDown) return;
    const point = d3.pointer(e, svgRef);
    if (controlledPointRef) {
      controlledPointRef.set(point);
    }
  }}
  on:mouseup={() => {
    if (!mouseDown) return;
    mouseDown = false;
    center();
  }}
  bind:innerWidth={windowWidth}
  bind:innerHeight={windowHeight}
/>

<Header bind:hideControls {randomise} />
<div class="center">
  <div class="svg-root">
    <svg
      width="100%"
      height="100%"
      viewBox={`0 0 ${SVG_SIZE} ${SVG_SIZE}`}
      bind:this={svgRef}
    >
      <g bind:this={faceGroupRef}>
        <path
          id="eye-1"
          d="m -0.96218271,-7.0965548 c -0.17089999,0.00275 -0.34085599,0.018896 -0.50754279,0.052713 -0.2171771,0.044094 -0.4245,0.12823 -0.6366228,0.1923091 -0.5264458,0.25967 -0.8678686,0.6251535 -1.1064075,1.0721597 -0.2871968,0.18254 -0.5392416,0.4708599 -0.7156528,0.8824922 -0.5465626,1.1905099 -0.9894353,2.4066697 -1.2635874,3.6880266 -0.2477227,1.32724965 -0.3008669,2.664852 -0.08166,3.9962397 0.2385973,1.1489355 0.8643299,2.1122389 1.6332666,2.9732476 0.7879595,0.7179482 1.7456391,1.0901896 2.77392291,1.2995888 C 0.47626256,7.26346 1.3783835,6.5970657 2.3640668,5.8168329 3.3196606,4.9610464 3.915708,3.8471622 4.4153099,2.6863966 4.8211581,1.7239674 5.147442,0.74521508 5.3364391,-0.28246477 5.5599179,-1.3893021 5.259979,-2.4301061 4.7630395,-3.4102556 4.2574229,-4.256543 3.5859217,-4.9872611 2.8645848,-5.6529328 2.1773739,-6.2387119 1.4353126,-6.6730813 0.56747186,-6.9235408 0.0745436,-7.0018423 -0.44948303,-7.1062411 -0.96217965,-7.0965216 Z"
          fill={SMILE_COLOR}
          stroke="none"
          transform={`translate(${$eye1Point[0]}, ${$eye1Point[1]})`}
        />
        <path
          id="eye-2"
          d="m 0.45694464,-7.2983722 c -0.06069,-0.0013 -0.121832,-0.0011 -0.183528,8.772e-4 -0.06577,0.00217 -0.131501,0.00608 -0.198458,0.012303 -0.95115,0.02685 -1.91312504,0.072778 -2.81781704,0.398658 -0.997911,0.4336762 -1.880337,1.038975 -2.465699,1.9827503 -0.486556,0.844947 -0.76794,1.718625 -0.689309,2.704541 -0.0032,0.730897 -0.0087,1.46173299 8.79e-4,2.19261099 0.02226,1.01593601 0.05438,2.04484201 0.361777,3.02240901 0.341647,0.907244 0.803307,1.761801 1.378613,2.54384 0.270138,0.337687 0.577414,0.544658 0.891272,0.648047 0.133802,0.120139 0.292367,0.229842 0.480317,0.324029 0.354773,0.177765 0.144499,0.07521 0.633986,0.302048 0.727581,0.253968 1.45535004,0.508707 2.24178704,0.459249 0.700496,-2.1e-5 1.40555896,0.0061 2.09777696,-0.117678 0.885495,-0.214204 1.675901,-0.60549 2.210169,-1.375108 0.603737,-0.744274 1.109167,-1.548763 1.345246,-2.490279 0.189998,-0.931195 0.230654,-1.861853 0.05529,-2.80201801 -0.11817,-0.696413 -0.308314,-1.374389 -0.426741,-2.07054699 -0.175955,-0.801807 -0.262026,-1.629164 -0.594475,-2.38836 -0.41205,-0.967467 -0.886647,-1.9038324 -1.820294,-2.4736021 -0.784671,-0.4676898 -1.590064,-0.853564 -2.50082696,-0.8737053 z"
          fill={SMILE_COLOR}
          stroke="none"
          transform={`translate(${$eye2Point[0]}, ${$eye2Point[1]})`}
        />
        <path
          id="mouth"
          stroke={SMILE_COLOR}
          d={curveLine}
          stroke-width="4"
          fill="none"
          stroke-linejoin="round"
          stroke-linecap="round"
        />
      </g>

      {#if !hideControls}
        <g class="controls">
          {#if bbox}
            <rect
              x={bbox.x}
              y={bbox.y}
              width={bbox.width}
              height={bbox.height}
              fill="none"
              stroke="red"
              stroke-width={s(1)}
            />
          {/if}

          <path
            stroke="red"
            vector-effect="non-scaling-stroke"
            d={line}
            stroke-width="1"
            fill="none"
          />

          <Anchor
            point={mouthPoint1}
            {s}
            bind:controlledPointRef
            bind:mouseDown
          />
          <Anchor
            point={mouthPoint2}
            {s}
            bind:controlledPointRef
            bind:mouseDown
          />
          <Anchor
            point={mouthPoint3}
            {s}
            bind:controlledPointRef
            bind:mouseDown
          />
          <Anchor
            point={mouthPoint4}
            {s}
            bind:controlledPointRef
            bind:mouseDown
          />
        </g>
      {/if}
    </svg>
  </div>
</div>

<style lang="scss">
  .center {
    display: flex;
    justify-content: center;
    width: 100%;
    height: 100vh;
    align-items: center;
  }

  .svg-root {
    display: flex;
    position: relative;
    width: 100%;
    height: 100%;
  }
</style>
