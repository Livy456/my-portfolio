<svelte:head>
    <title>Projects</title>
</svelte:head>
<script> 

import projects from '$lib/projects.json';
import Project from '$lib/Projects.svelte';
import Pie from '$lib/Pie.svelte';
import * as d3 from 'd3';

let selectedYearIndex = -1;
let query = "";
let filterProjects;

$: filterProjects = projects.filter( (project) => {
    
    let values = Object.values(project).join("\n").toLowerCase();
    return values.includes(query.toLowerCase());

});

let pieData;

$:{
    pieData = {};

    // gets an array of arrays-> [year, number of projects for that year]
    let rolledData = d3.rollups(
    filterProjects,
    (v) => v.length,
    (d) => d.year
    );

    pieData = rolledData.map( ([year, count]) => {
    return {value: count, label: year};
    });
}

</script>

<style>
input.search-bar{
    background-color: rgb(255, 255, 255);
    font-size: 30px;
    width: 700px;
    height: 50px;
}

</style>

<h1>{projects.length} Projects</h1>
<!-- Debugging for importing json file for projects page -->
<!-- <pre>{JSON.stringify(projects, null, "\t") }</pre> -->
<Pie data={pieData} bind:selectedIndex="{selectedYearIndex}"/>   

    <input 
        class="search-bar"
        type="search"
        bind:value={query}
        aria-label="Search Projects"
        placeholder="🔍 Search projects…"
    />

    <div class="projects">
        {#each filterProjects as p}
            <Project info={p}/>
        {/each}
    </div>