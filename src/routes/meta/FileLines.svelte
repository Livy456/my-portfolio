<script>
    import * as d3 from 'd3';
    import { flip as originalFlip} from "svelte/animate";
    import { get } from 'svelte/store';
    import { scale } from 'svelte/transition';
    export let lines = {}; 
    export let colors = d3.scaleOrdinal(d3.schemeTableau10);
    
    let files = [];
    $: {
        files = d3.groups(lines, d => d.file).map( ([name, lines]) => {
            return {name, lines};
        });
        files = d3.sort(files, d => -d.lines.length);
    }
    $: flip = getFlip(files);

    function getFlip ()
    {
        return originalFlip;
    }

</script>

<style>

div{
    grid-column: 1 / -1;
    display: grid;
    grid-template-columns: 3fr 10fr;
    background: hsl(0 0% 100% / 90%);
    box-shadow: 0 0 .2em .2em hsl(0 0% 100% / 90%);

    dd{
        grid-column: 2;
        display: flex;
        flex-wrap: wrap;
        align-items: start;
        align-content: start;
        gap: .15em;
        padding-top: .6em;
        margin-left: 0;
    }
    dt{
        grid-column: 1;
    }
}

.line{
    display: flex;
    width: .5em;
    aspect-ratio: 1;
    background-color: var(--color);
    border-radius: 50%;
}

small{
    display: block;
    color: gray;
}

</style>

<dl class="files">
    {#each files as file (file.name)}
        <div animate:flip={ {delay:10, duration:500}}>
            <dt>
                <code>{file.name}</code>
                <small>{file.lines.length} lines</small>
            </dt>
            <dd>
                {#each file.lines as line (line.line)}
                    <div in:scale class="line" style="--color: {colors(line.type)}"></div>
                {/each}
            </dd>
            
        </div>

    {/each}

</dl>

