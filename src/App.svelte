<script lang="ts">
    import Button from '@smui/button'
    import download from 'downloadjs'

    let videoRef:(HTMLVideoElement | null)
    let canvasForSave:(HTMLCanvasElement | null)
    let videoSize: { width: number, height: number} = { width: 0, height: 0}
    let isCapturing = false

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
            const { width, height} = currentTrack.getSettings()
            videoSize.width = width
            videoSize.height = height
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

    const saveCapture = () => {
        if(!videoRef || !canvasForSave) return
        canvasForSave.width = videoSize.width
        canvasForSave.height = videoSize.height
        const ctx = canvasForSave.getContext('2d')
        ctx.clearRect(0, 0, videoSize.width, videoSize.height)
        ctx.drawImage(videoRef, 0, 0)
        const data = canvasForSave.toDataURL("image/jpeg", 1)
        download(data, "output.jpg", "image/jpeg")
    }
</script>

<main>
    <div class='button-wrapper'>
        {#if isCapturing}
            <Button on:click={stopCapture} variant='raised'>
                キャプチャーを終了
            </Button>
        {:else}
            <Button on:click={startCapture} variant='raised'>
                画面キャプチャーを開始する
            </Button>
        {/if}
        <Button on:click={saveCapture} variant='raised'>
            キャプチャーを保存
        </Button>
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
