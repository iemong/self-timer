<script lang="ts">
    import Button from '@smui/button'
    import download from 'downloadjs'

    let videoRef:(HTMLVideoElement | null)
    let canvasForSave:(HTMLCanvasElement | null)
    let isCapturing = false
    let currentVideoTrack:(MediaStreamTrack | null) = null

    const displayMediaOptions:MediaStreamConstraints = {
        video: true,
        audio: false
    }

    const startCapture = async () => {
        try {
            if(!videoRef) return
            const stream = await navigator.mediaDevices.getDisplayMedia(displayMediaOptions)
            videoRef.srcObject = stream

            isCapturing = true
            const currentTrack = stream.getVideoTracks()[0]
            currentTrack.onended = () => {
                stopCapture()
            };
            currentVideoTrack = currentTrack
        } catch(e) {
            console.error(e)
        }
    }

    const stopCapture = () => {
        if ('getTracks' in videoRef.srcObject) {
            const tracks = videoRef.srcObject.getTracks()
            tracks.forEach( track => track.stop())
            videoRef.srcObject = null
            isCapturing = false
        }
    }

    const MAX_COUNT = 5
    let count = MAX_COUNT
    const countdown = () => {
        setTimeout(() => {
            count--
            console.log(count)
            if(count > 0) {
                countdown()
            } else {
                saveCapture()
                resetCount()
            }
        }, 1000)
    }

    const resetCount = () => {
        count = MAX_COUNT
    }


    const saveCapture = async () => {
        try {
            if(!canvasForSave) return
            const imageCapture = new ImageCapture(currentVideoTrack);
            await imageCapture.getPhotoCapabilities()
            const { width, height} = currentVideoTrack.getSettings()
            const bitmap = await imageCapture.grabFrame()
            canvasForSave.width = bitmap.width;
            canvasForSave.height = bitmap.height;
            const context = canvasForSave.getContext('2d')
            context.drawImage(bitmap, 0, 0);
            download(canvasForSave.toDataURL("image/jpeg", 1), "output.jpg", "image/jpeg")
            context.clearRect(0, 0, bitmap.width, bitmap.height);
        } catch (e) {
            console.error(e)
        }
    }
</script>

<main>
    <div class='button-wrapper'>
        {#if isCapturing}
            <Button on:click={stopCapture} variant='raised'>
                キャプチャーを終了
            </Button>
            <Button on:click={countdown} variant='raised'>
                キャプチャーを保存
            </Button>
        {:else}
            <Button on:click={startCapture} variant='raised'>
                画面キャプチャーを開始する
            </Button>
        {/if}
    </div>
    <div class='capture-wrapper'>
        {#if isCapturing}
            <p class='attention'>この画面が保存されます。</p>
        {/if}
        <video bind:this={videoRef} autoplay class='capture'>
            <track kind="captions" src=''>
        </video>
        <canvas bind:this={canvasForSave} class='hide'></canvas>
    </div>
</main>

<style>
    .capture-wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    .capture {
        max-width: 80vw;
        max-height: 80vh;
    }
    .attention {
        font-weight: bold;
        margin-bottom: 16px;
    }
    .hide {
        display: none;
    }
</style>
