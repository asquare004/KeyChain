<template>
  <div class="flex flex-col bg-gradient-to-r from-blue-500 via-purple-500 to-indigo-600 items-center min-h-screen   p-4 bg-gray-100">
    <h1 class="mb-4 text-gray-900 text-3xl font-bold">
      {{ isVerifying ? 'Verification' : 'Login/Register' }}
    </h1>

    <div class="flex flex-col lg:flex-row w-full max-w-6xl bg-gray-50 rounded-lg shadow-md">
      
      <div class="lg:w-[90vw] p-4 flex flex-col justify-center">
        <h2 class="text-xl font-bold mb-4">About Our Website</h2>
        <p class="text-gray-600">
          Welcome to our platform! We provide a seamless experience for users to verify their identities and access
          various services. Our goal is to ensure security and convenience for all our users. Please follow the
          instructions to complete your verification.
        </p>
        <p class="text-gray-600 mt-4">
          Feel free to reach out to our support team if you have any questions or need assistance.
        </p>
      </div>

      <!-- Verification/Login Section (Right) -->
      <div class="lg:w-1/2 p-6">
        <div>
          <div class="[--clr:#1f1f1f] dark:[--clr:#999999] mt-12 relative flex flex-row items-center">
            <input v-model="username" value="" name="email" required="" aria-invalid="false" placeholder=""
              spellcheck="false" autocomplete="off" id="email" type="email"
              class="peer text-black dark:text-white pl-2 p-2 h-[40px] min-h-[40px] pr-[40px] leading-normal appearance-none resize-none box-border text-base w-full text-inherit block text-left border border-solid bg-white dark:bg-zinc-800 rounded-[10px] m-0 p-0 outline-0 focus-visible:outline-0 focus-visible:border-teal-500 focus-visible:outline-none focus-visible:ring-4 focus-visible:ring-[#71717a2e] dark:focus-visible:ring-[#14b8a61a]" />
            <label
              class="cursor-text text-[--clr] inline-block z-0 text-lg mb-px font-semibold  text-start select-none absolute duration-300 transform origin-[0] translate-x-[32px] peer-focus-visible:text-teal-500 peer-focus-visible:translate-x-[8px] peer-[:not(:placeholder-shown)]:translate-x-[8px] peer-focus-visible:translate-y-[-36px] peer-[:not(:placeholder-shown)]:translate-y-[-36px] peer-[:not(:placeholder-shown)]:text-[-36px]"
              for="email">
              Address
            </label>
            <span
              class="pointer-events-none absolute z-[+1] left-0 top-0 bottom-0 flex items-center justify-center size-[40px] text-[--clr] peer-focus-visible:hidden peer-[:not(:placeholder-shown)]:hidden">
              <svg xmlns="http://www.w3.org/2000/svg" width="1rem" height="1rem" stroke-linejoin="round"
                stroke-linecap="round" viewBox="0 0 24 24" stroke-width="2" fill="none" stroke="currentColor">
                <path fill="none" d="M0 0h24v24H0z" stroke="none"></path>
                <path d="M12 12m-4 0a4 4 0 1 0 8 0a4 4 0 1 0 -8 0"></path>
                <path d="M16 12v1.5a2.5 2.5 0 0 0 5 0v-1.5a9 9 0 1 0 -5.5 8.28"></path>
              </svg>
            </span>
            <div
              class="group w-[40px] absolute top-0 bottom-0 right-0 flex items-center justify-center text-[--clr] peer-focus-visible:text-teal-600 peer-focus-visible:[&amp;_span]:border-teal-600">
              <svg xmlns="http://www.w3.org/2000/svg" width="1rem" height="1rem" stroke-linejoin="round"
                stroke-linecap="round" viewBox="0 0 24 24" stroke-width="2" fill="none" stroke="currentColor">
                <path fill="none" d="M0 0h24v24H0z" stroke="none"></path>
                <path d="M3 12a9 9 0 1 0 18 0a9 9 0 0 0 -18 0"></path>
                <path d="M12 8v4"></path>
                <path d="M12 16h.01"></path>
              </svg>
              <span
                class="text-sm absolute cursor-default select-none rounded-[4px] px-1.5 opacity-0 right-0 -z-10 transition-all duration-300 group-hover:opacity-100 group-hover:-translate-y-[calc(100%+18px)]">Required!</span>
            </div>
          </div>

          <div class="relative flex flex-col items-center">
            <video ref="recordedVideo" :src="videoUrl" controls class="w-full h-auto rounded-md border border-gray-300 m-3"
              v-if="videoUrl"></video>

           

            <button @click="startRecording" v-if="!recording && !videoUrl"
              class="  p-2 mb-4 mt-4 text-black font-semibold rounded-3xl">Start Recording</button>


            <p v-if="recording" class="m-3">Recording: {{ recordingTime }} seconds</p>
          </div>
 <div class="flex">
          <button 
            @click="submitForm" 
            class="m-4 p-2 mx-auto rounded-3xl" 
            v-if="videoUrl">
            Final Submit
          </button>
          </div>
  <div class="flex">
          <button 
            @click="toggleView" 
            class="p-2 text-blue-600 mx-auto font-semibold rounded-3xl">
            Go to {{ isVerifying ? 'Registration' : 'Verification' }}
          </button>
</div>
          <!-- Error Message -->
          <p v-if="errorMessage" class="mt-4 text-red-600">{{ errorMessage }}</p>
        </div>
      </div>
    </div>

    <div v-if="submitted" class="mt-4">
      <p class="text-green-600">Form submitted:</p>
      <pre class="bg-gray-100 p-2 rounded">{{ JSON.stringify(formData, null, 2) }}</pre>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      recording: false,
      isVerifying: false,
      formData: {},
      submitted: false,
      mediaRecorder: null,
      recordedChunks: [],
      videoUrl: '',
      stream: null,
      recordingTime: 0, // To hold the recording time
      recordingInterval: null, // To hold the interval reference
      errorMessage: '', // To hold error messages
    };
  },
  methods: {
    stopRecording() {
      if (this.mediaRecorder && this.recording) {
        this.mediaRecorder.stop();
        this.recording = false;
      }
    },
    async startStream() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.stream = stream;
        this.$refs.video.srcObject = stream;
      } catch (error) {
        console.error('Error accessing media devices.', error);
        alert('Unable to access the camera.');
      }
    },
    async startRecording() {
      await this.startStream();
      if (!this.stream) return;

      this.recordedChunks = [];
      this.mediaRecorder = new MediaRecorder(this.stream);

      this.mediaRecorder.ondataavailable = (event) => {
        this.recordedChunks.push(event.data);
      };

      this.mediaRecorder.onstop = async () => {
        const videoBlob = new Blob(this.recordedChunks, { type: 'video/webm' });
        this.videoUrl = URL.createObjectURL(videoBlob);
        const base64Video = await this.convertBlobToBase64(videoBlob);
        this.formData.videoBase64 = base64Video; // Store the Base64 video data
        this.videoUrl = URL.createObjectURL(videoBlob);
        this.recording = false; // Stop recording
        clearInterval(this.recordingInterval); // Clear the interval when recording stops
        // Stop the video stream to turn off the camera
        if (this.stream) {
          this.stream.getTracks().forEach(track => track.stop()); // Stop all tracks (video/audio)
        }
      };

      this.mediaRecorder.start();
      this.recording = true;

      // Start recording timer
      this.recordingTime = 0;
      this.recordingInterval = setInterval(() => {
        this.recordingTime++;
      }, 1000);

      // Automatically stop after 5 seconds
      setTimeout(() => {
        this.stopRecording(); // Ensure this function is properly defined and accessible
      }, 5000);
    },

    // Helper function to convert Blob to Base64
    convertBlobToBase64(blob) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onloadend = () => resolve(reader.result);
        reader.onerror = reject;
        reader.readAsDataURL(blob);
      });
    },

    submitForm() {
      console.log(this.username, this.formData.videoBase64);
      if (!this.username || !this.formData.videoBase64) {
        this.errorMessage = 'Both username and recorded video are required for submission.';
        return;
      }

      this.errorMessage = '';

      // Include the Base64 video in the formData object
      this.formData = {
        username: this.username,
        videoBase64: this.formData.videoBase64,
      };

      console.log(this.formData); // Log to see the Base64 data
      this.submitted = true;
    },
  },
};

</script>

<style>
/* From Uiverse.io by xueyuantan */
button {
  width: full;
  height: 3em;
  border-radius: 30em;
  font-size: 15px;
  font-family: inherit;
  border: none;
  position: relative;
  overflow: hidden;
  z-index: 1;
  box-shadow: 6px 6px 12px #c5c5c5,
    -6px -6px 12px #ffffff;
}

button::before {
  content: '';
  width: 0;
  height: 3em;
  border-radius: 30em;
  position: absolute;
  top: 0;
  left: 0;
  background-image: linear-gradient(to right, #0fd850 0%, #f9f047 100%);
  transition: .5s ease;
  display: block;
  z-index: -1;
}

button:hover::before {
  width: 9em;
}

/* Add any additional custom styles here */
</style>
