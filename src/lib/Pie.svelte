<!-- <p>This is the text from Pie.svelte</p> -->
<style>
    svg{
        max-width: 20em;
        margin-block: 2em;
        /* Doesn't clip the shape even if it'd outside of viewbox */
        overflow: visible;
    }
</style>


<svg viewBox= "-50 -50 100 100">
    <circle cx="0" cy="0" r="50" fill="red">
    <!-- Circle repeatedly gets printed 12 times;
    due to /src/projects/page.svelte that repeatedly prints
    data from the /src/lib/Projects.svelte page -->
</svg>
