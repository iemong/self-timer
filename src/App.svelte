<script lang="ts">
    import { fly } from 'svelte/transition'
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
        if (!('getTracks' in videoRef.srcObject)) return
        const tracks = videoRef.srcObject.getTracks()
        tracks.forEach(track => track.stop())
        videoRef.srcObject = null
        isCapturing = false
    }

    const MAX_COUNT = 3
    let count = MAX_COUNT
    let isCounting = false
    const handleClickCapture = () => {
        if(isCounting) return
        resetCount()
        countdown()
        isCounting = true
    }
    const countdown = () => {
        setTimeout(() => {
            count--
            if(count > 0) {
                countdown()
            } else {
                saveCapture()
                isCounting = false
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

<main class='main'>
    {#if !isCapturing}
        <div class='prepare-button-wrapper'>
            <Button on:click={startCapture} variant='raised'>
                保存したい画面を選択する
            </Button>
        </div>
    {/if}
    <div class='capture-wrapper'>
        {#if isCapturing}
            <div class='ready-button-wrapper'>
                <Button class='save-button' on:click={handleClickCapture} variant='raised'>
                    キャプチャーを保存する
                </Button>
                <Button class='end-button' on:click={stopCapture} variant='outlined'>
                    やり直す
                </Button>
            </div>
            <div class='mdc-typography--body1'>
                <strong class='attention'>↓↓↓この画面が保存されます。↓↓↓</strong>
            </div>
        {/if}
        <div class='video-wrapper'>
            <video bind:this={videoRef} autoplay class='capture' data-is-capturing={isCapturing}>
                <track kind="captions" src=''>
            </video>
            {#if isCapturing && isCounting}
                {#key count}
                    <strong in:fly={{ y: -20 }} class='counter'>{count}</strong>
                {/key}
            {/if}
        </div>
        <canvas bind:this={canvasForSave} class='hide'></canvas>
    </div>
</main>

<style lang='scss'>
    .main {
        height: 100%;
    }
    .prepare-button-wrapper {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        height: 100%;
    }
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
        display: block;
        font-weight: bold;
        margin-bottom: 16px;
    }
    .video-wrapper {
        position: relative;
    }
    .capture {
        display: block;
        margin: 0 auto;
        &[data-is-capturing="false"] {
             height: 0;
         }
    }
    .ready-button-wrapper {
        display: flex;
        justify-content: space-between;
        gap: 0 16px;
        margin: 32px 0 48px;
    }
    .counter {
        position: absolute;
        display: block;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        min-width: 1em;
        height: 1em;
        text-align: center;
        margin: auto;
        color: white;
        --length: 1px;
        text-shadow:
                var(--length) 0 0 black,
                0 var(--length) 0 black,
                calc(var(--length) * -1) 0 0 black,
                0 calc(var(--length) * -1) 0 black;
        font-size: 128px;
        font-weight: bold;
    }
    .hide {
        display: none;
    }
</style>
