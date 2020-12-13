<script>
    import { quintOut } from 'svelte/easing';
    import { crossfade } from 'svelte/transition';

    export let keys = [];
    export let formation = 1;

    const [send, receive] = crossfade({
        duration: (d) => Math.sqrt(d * 200),

        fallback(node, params) {
            const style = getComputedStyle(node);
            const transform = style.transform === 'none' ? '' : style.transform;

            return {
                duration: 600,
                easing: quintOut,
                css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`,
            };
        },
    });
</script>

<style>
    div {
        border-radius: 50%;
        width: 3vmin;
        height: 3vmin;
        background-color: yellow;
        display: inline-block;
    }
</style>

{#each keys as key}
    <div class="ball" in:receive={{ key }} out:send={{ key }} />
{/each}
