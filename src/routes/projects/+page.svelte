<svelte:head>
    <title>Projects</title>
</svelte:head>
<script> 

import projects from '$lib/projects.json';
import Project from '$lib/Projects.svelte';
import Pie from '$lib/Pie.svelte';
import * as d3 from 'd3';

let selectedYearIndex = -1;
let selectedYear;
let query = "";
let filterProjects;
let filteredYear;

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

// filter function iterates through each element in array
$: filteredYear = filterProjects.filter( (project) => {
    
    selectedYear =  selectedYearIndex > -1 ? pieData[selectedYearIndex].label : null;
    return project.year === selectedYear; 
});

</script>

<style>
input.search-bar{
    /* background-color: rgb(255, 255, 255); */
    font-size: 30px;
    width: 1240px;
    height: 50px;
}

</style>

<h1>{projects.length} Projects</h1>
<!-- Debugging for importing json file for projects page -->
<!-- <pre>{JSON.stringify(projects, null, "\t") }</pre> -->
<Pie data={pieData} bind:selectedIndex="{selectedYearIndex}"/>   

    <!-- {selectedYearIndex} -->
    <input 
        class="search-bar"
        type="search"
        bind:value={query}
        aria-label="Search Projects"
        placeholder="🔍 Search projects…"
    />

    <div class="projects">

        {#if filteredYear}
            {#each filteredYear as p}
            <Project info={p}/>
            {/each}
        {/if}

        {#if filteredYear.length === 0}
            {#each filterProjects as p}
                <Project info={p}/>
            {/each}
        {/if}
    </div>