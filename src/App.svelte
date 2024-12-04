<script lang="ts" context="module">
  const SVG_SIZE = 50;
</script>

<script lang="ts">
  import * as d3 from "d3";

  const curveLineGen = d3
    .line()
    .x((d) => d[0])
    .y((d) => d[1])
    .curve(d3.curveBasis);

  const lineGen = d3
    .line()
    .x((d) => d[0])
    .y((d) => d[1]);

  const s = d3.scaleLinear().domain([0, SVG_SIZE]).range([0, 500]);

  let lineHovered = false;
  $: console.log(lineHovered);

  $: mouthPoint1 = [6.866, 23.666];
  $: mouthPoint2 = [9.117, 41.293];
  $: mouthPoint3 = [41.918, 40.537];
  $: mouthPoint4 = [43.134, 23.552];

  $: points = [mouthPoint1, mouthPoint2, mouthPoint3, mouthPoint4] as [
    number,
    number,
  ][];

  $: curveLine = curveLineGen(points);

  $: line = lineGen(points);
</script>

<div class="center">
  <div class="svg-root">
    <svg width="500" height="500" viewBox={`0 0 ${SVG_SIZE} ${SVG_SIZE}`}>
      <path
        id="eye-1"
        d="m -0.96218271,-7.0965548 c -0.17089999,0.00275 -0.34085599,0.018896 -0.50754279,0.052713 -0.2171771,0.044094 -0.4245,0.12823 -0.6366228,0.1923091 -0.5264458,0.25967 -0.8678686,0.6251535 -1.1064075,1.0721597 -0.2871968,0.18254 -0.5392416,0.4708599 -0.7156528,0.8824922 -0.5465626,1.1905099 -0.9894353,2.4066697 -1.2635874,3.6880266 -0.2477227,1.32724965 -0.3008669,2.664852 -0.08166,3.9962397 0.2385973,1.1489355 0.8643299,2.1122389 1.6332666,2.9732476 0.7879595,0.7179482 1.7456391,1.0901896 2.77392291,1.2995888 C 0.47626256,7.26346 1.3783835,6.5970657 2.3640668,5.8168329 3.3196606,4.9610464 3.915708,3.8471622 4.4153099,2.6863966 4.8211581,1.7239674 5.147442,0.74521508 5.3364391,-0.28246477 5.5599179,-1.3893021 5.259979,-2.4301061 4.7630395,-3.4102556 4.2574229,-4.256543 3.5859217,-4.9872611 2.8645848,-5.6529328 2.1773739,-6.2387119 1.4353126,-6.6730813 0.56747186,-6.9235408 0.0745436,-7.0018423 -0.44948303,-7.1062411 -0.96217965,-7.0965216 Z"
        fill="black"
        stroke="none"
        transform={`translate(${mouthPoint1[0] + 10}, ${mouthPoint1[1] - 10})`}
      />
      <path
        id="eye-2"
        d="m 0.45694464,-7.2983722 c -0.06069,-0.0013 -0.121832,-0.0011 -0.183528,8.772e-4 -0.06577,0.00217 -0.131501,0.00608 -0.198458,0.012303 -0.95115,0.02685 -1.91312504,0.072778 -2.81781704,0.398658 -0.997911,0.4336762 -1.880337,1.038975 -2.465699,1.9827503 -0.486556,0.844947 -0.76794,1.718625 -0.689309,2.704541 -0.0032,0.730897 -0.0087,1.46173299 8.79e-4,2.19261099 0.02226,1.01593601 0.05438,2.04484201 0.361777,3.02240901 0.341647,0.907244 0.803307,1.761801 1.378613,2.54384 0.270138,0.337687 0.577414,0.544658 0.891272,0.648047 0.133802,0.120139 0.292367,0.229842 0.480317,0.324029 0.354773,0.177765 0.144499,0.07521 0.633986,0.302048 0.727581,0.253968 1.45535004,0.508707 2.24178704,0.459249 0.700496,-2.1e-5 1.40555896,0.0061 2.09777696,-0.117678 0.885495,-0.214204 1.675901,-0.60549 2.210169,-1.375108 0.603737,-0.744274 1.109167,-1.548763 1.345246,-2.490279 0.189998,-0.931195 0.230654,-1.861853 0.05529,-2.80201801 -0.11817,-0.696413 -0.308314,-1.374389 -0.426741,-2.07054699 -0.175955,-0.801807 -0.262026,-1.629164 -0.594475,-2.38836 -0.41205,-0.967467 -0.886647,-1.9038324 -1.820294,-2.4736021 -0.784671,-0.4676898 -1.590064,-0.853564 -2.50082696,-0.8737053 z"
        fill="black"
        stroke="none"
        transform={`translate(${mouthPoint4[0] - 10}, ${mouthPoint4[1] - 10})`}
      />
      <path
        id="mouth"
        stroke="black"
        d={curveLine}
        stroke-width="4"
        fill="none"
        stroke-linejoin="round"
        stroke-linecap="round"
        on:mouseover={() => (lineHovered = true)}
        on:mouseout={() => (lineHovered = false)}
      />

      <g class="debug">
        {#if lineHovered}
          <path
            stroke="red"
            vector-effect="non-scaling-stroke"
            d={line}
            stroke-width="2"
            fill="none"
          />
        {/if}
      </g>
    </svg>

    <div class="controls">
      <div
        class="anchor"
        style:transform={`translate(-50%, -50%) translate(${s(mouthPoint1[0])}px, ${s(mouthPoint1[1])}px)`}
        on:click={() => console.log("hmm")}
      ></div>
    </div>

    <!-- <svg
      width="500"
      height="500"
      class="svg-interactive"
      class:show-anchors={lineHovered}
    >
      <circle
        stroke="red"
        class="anchor"
        cx={s(mouthPoint1[0])}
        cy={s(mouthPoint1[1])}
        r="5"
        stroke-width="2"
        fill="white"
        vector-effect="non-scaling-size"
      />
      <circle
        stroke="red"
        class="anchor"
        cx={s(mouthPoint2[0])}
        cy={s(mouthPoint2[1])}
        r="5"
        stroke-width="2"
        fill="white"
        vector-effect="non-scaling-size"
      />
      <circle
        stroke="red"
        class="anchor"
        cx={s(mouthPoint3[0])}
        cy={s(mouthPoint3[1])}
        r="5"
        stroke-width="2"
        fill="white"
        vector-effect="non-scaling-size"
      />
    </svg> -->
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
    border: 1px solid black;
    display: flex;
    position: relative;
  }

  .controls {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
  }

  .anchor {
    cursor: pointer;
    position: absolute;
    left: 0;
    top: 0;
    width: 10px;
    height: 10px;
    background-color: red;
    border-radius: 9999px;
    pointer-events: all;
  }

  .controls {
    &.show-anchors {
      .anchor {
        opacity: 1;
      }
    }
  }

  .debug {
    pointer-events: none;
  }
</style>
