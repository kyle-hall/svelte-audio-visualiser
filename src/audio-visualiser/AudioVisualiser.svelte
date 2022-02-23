<script>
  import { onMount } from 'svelte';

  let root;

  onMount(() => {
    const audioElement = document.querySelector('audio')
    const audioContext = new (window.AudioContext || window.webkitAudioContext)()

    const analyserNode = audioContext.createAnalyser()
    analyserNode.fftSize = 2048
    analyserNode.maxDecibels = -25
    analyserNode.minDecibels = -60
    analyserNode.smoothingTimeConstant = 0.5

    const track = audioContext.createMediaElementSource(audioElement)

    track.connect(analyserNode)
    analyserNode.connect(audioContext.destination)

    // I don't think this is doing anything because there's no data from the track
    // and it's not waiting to receive it before filling the array
    const bufferLength = analyserNode.frequencyBinCount
    const dataArray = new Uint8Array(bufferLength)
    analyserNode.getByteTimeDomainData(dataArray)

    const canvas = document.querySelector("#oscilloscope");
    const canvasCtx = canvas.getContext("2d");

    // draw an oscilloscope of the current audio source

    function draw() {

      console.log('calling draw')

      requestAnimationFrame(draw);

      analyserNode.getByteTimeDomainData(dataArray);

      canvasCtx.fillStyle = "rgb(200, 200, 200)";
      canvasCtx.fillRect(0, 0, canvas.width, canvas.height);

      canvasCtx.lineWidth = 2;
      canvasCtx.strokeStyle = "rgb(256, 0, 0)";

      canvasCtx.beginPath();

      var sliceWidth = canvas.width * 1.0 / bufferLength;
      var x = 0;

      for (var i = 0; i < bufferLength; i++) {
        console.log('drawing a thing')
        var v = dataArray[i] / 128.0;
        var y = v * canvas.height / 2;

        if (i === 0) {
          canvasCtx.moveTo(x, y);
        } else {
          canvasCtx.lineTo(x, y);
        }

        x += sliceWidth;
      }

      canvasCtx.lineTo(canvas.width, canvas.height / 2);
      canvasCtx.stroke();
    }

    draw();
  });

</script>

<div bind:this={root} class="visualiser-container">
  <div class="visualiser">
    <h2>Visualiser</h2>
    <canvas id="oscilloscope"></canvas>
    <audio controls>
      <source src="whereTheWavesTakeUs.mp3" type="audio/mpeg">
    </audio>
  </div>
</div>

<style>
  .visualiser {
    display: grid;
    place-content: center;

    height: 100vh;
    width: 100vw;
  }
</style>
