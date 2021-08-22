<script lang="ts">
    import Button from '@smui/button';

    let count: number = 0
    const increment = () => {
        count += 1
    }
</script>

<Button on:click={increment} variant="raised">
    Clicks: {count}
</Button>

<style>
</style>
