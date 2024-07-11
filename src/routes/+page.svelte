<script>
  import * as faceapi from 'face-api.js';
  import { onMount } from 'svelte';

  let videoElement;
  let mood = 'N/A';

  async function startWebcam() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      videoElement.srcObject = stream;
      await videoElement.play();
      startEmotionDetection();
    } catch (err) {
      console.error('Error accessing webcam: ', err);
    }
  }

  async function loadModels() {
    const MODEL_URL = 'public/models';
    console.log('Loading models...');
    try {
      await Promise.all([
        faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL),
        faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL),
        faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL),
        faceapi.nets.faceExpressionNet.loadFromUri(MODEL_URL)
      ]);
      console.log('Models loaded successfully');
    } catch (error) {
      console.error('Error loading models:', error);
    }
  }

  async function startEmotionDetection() {
    await loadModels();
    setInterval(async () => {
      const detections = await faceapi
        .detectAllFaces(videoElement, new faceapi.TinyFaceDetectorOptions())
        .withFaceExpressions();
      console.log('Detections:', detections);
      if (detections.length > 0) {
        mood = getDominantEmotion(detections[0].expressions);
        console.log('Mood detected:', mood);
      } else {
        mood = 'N/A';
      }
    }, 1000);
  }

  function getDominantEmotion(expressions) {
    return Object.keys(expressions).reduce((a, b) => (expressions[a] > expressions[b] ? a : b));
  }

  onMount(() => {
    startWebcam();
  });
</script>

<main>
  <h1>Webcam Live Feed</h1>
  <video bind:this={videoElement} autoplay />
  <p>Mood: {mood}</p>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    background-color: #f0f0f0;
    font-family: Arial, sans-serif;
  }

  h1 {
    margin-bottom: 20px;
  }

  video {
    border: 2px solid #ccc;
    border-radius: 8px;
    width: 80%;
    height: auto;
    background-color: black;
  }

  p {
    margin-top: 20px;
    font-size: 24px;
  }
</style>
