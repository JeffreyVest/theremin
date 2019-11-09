<script>
  import * as tmImage from "@teachablemachine/image";
  import { onMount } from "svelte";
  import _ from "lodash";
  import Tone from "tone";

  const URL = "https://teachablemachine.withgoogle.com/models/CBBvJbaj/";
  const modelURL = URL + "model.json";
  const metadataURL = URL + "metadata.json";

  let model;
  let webcam;
  let webcamContainer;
  let predictedClass = "";
  let note;
  let osc;
  let stopped = true;

  // Load the image model and setup the webcam
  onMount(async function() {
    model = await tmImage.load(modelURL, metadataURL);

    webcam = new tmImage.Webcam(200, 200, true);
    await webcam.setup();
    await webcam.play();
    window.requestAnimationFrame(loop);

    webcamContainer.appendChild(webcam.canvas);

    osc = new Tone.Oscillator(440, "sawtooth3").toMaster();
  });

  async function loop() {
    webcam.update(); // update the webcam frame
    await predict();
    window.requestAnimationFrame(loop);
  }

  // run the webcam image through the image model
  async function predict() {
    // predict can take in an image, video or canvas html element
    const prediction = await model.predict(webcam.canvas);
    predictedClass = _.maxBy(prediction, x => x.probability).className;
  }

  $: {
    if (predictedClass === "high") note = 500;
    else if (predictedClass === "mid") note = 400;
    else if (predictedClass === "low") note = 300;
  }

  $: if (osc) osc.frequency.linearRampTo(note, 0.1);

  $: {
    if (osc) {
      if (stopped) osc.stop();
      else osc.start();
    }
  }
</script>

<div bind:this={webcamContainer} />
{#if osc}
  <button on:click={() => (stopped = !stopped)}>
    {stopped ? 'Start' : 'Stop'}
  </button>
{/if}
