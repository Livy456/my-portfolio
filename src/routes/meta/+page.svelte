<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    let data = [];

    onMount(async() => {
        data = await d3.csv("loc.csv");
    });
</script>

<p>Total lines of code: {data.length}</p>