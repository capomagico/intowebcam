<script>
  import * as faceapi from 'face-api.js';
  import '@tensorflow/tfjs'; // Importa TensorFlow.js
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
    const MODEL_URL = '/models';
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
      if (detections.length > 0 && detections[0].expressions) {
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
  <h1 style="font: Helvetica; font-size: 100px; font-weight: 600;">INTO Cam</h1>
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
    background-color: #ffffff;
    font-family: Arial, sans-serif;
  }

  h1 {
    margin-bottom: 20px;
  }

  video {
    width: 80%;
    aspect-ratio: 16/9;
    height: auto;
    background-color: white;
  }

  p {
    margin-top: 20px;
    font-size: 24px;
  }
</style>
