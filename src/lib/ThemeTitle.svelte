<script lang="ts">
    import { fly } from 'svelte/transition'
    import { onDestroy, onMount } from 'svelte'
    export let style: { [key: string]: string | number } = null

    const titleList = [
        'みんなでピース',
        'みんなでファイティングボーズ',
        'みんなでガッツポーズ',
        '全員真顔',
        '自慢のポーズをヨロシク',
    ]
    const sample = <T>(array: T[]): T | undefined => {
        if (!array.length) return undefined
        return array[Math.floor(Math.random() * array.length)]
    }
    let title = sample(titleList)
    let prev_title = ''

    let count = 0
    let timer: number | null = null

    onMount(() => {
        timer = window.setInterval(() => {
            prev_title = title
            while (title === prev_title) {
                title = sample(titleList)
            }
            count++
            prev_title = title
        }, 5000)
    })
    onDestroy(() => {
        if (!timer) return
        clearInterval(timer)
        timer = null
    })
</script>

<div {style} class="theme-title">
    {#key count}
        <p in:fly={{ y: -20 }} class="text">{title}</p>
    {/key}
</div>

<style lang="scss">
    .theme-title {
        border: 2px solid;
        padding: 12px;
        border-radius: 8px;
        .text {
            font-size: 48px;
            font-weight: bold;
        }
    }
</style>
