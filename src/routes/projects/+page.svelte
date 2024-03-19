<svelte:head>
    <title>Projects</title>
</svelte:head>
<script> 

import projects from '$lib/projects.json';
import Project from '$lib/Projects.svelte';
import Pie from '$lib/Pie.svelte';
import * as d3 from 'd3';

// gets an array of arrays-> [year, number of projects for that year]
let rolledData = d3.rollups(
    projects,
    (v) => v.length,
    (d) => d.year
);

let pieData = rolledData.map( ([year, count]) => {
    return {value: count, label: year};
});

</script>

<h1>{projects.length} Projects</h1>
<!-- Debugging for importing json file for projects page -->
<!-- <pre>{JSON.stringify(projects, null, "\t") }</pre> -->
<Pie data={pieData}/>   

    <div class="projects">
        {#each projects as p}
            <Project info={p}/>
        {/each}
    </div>