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
let visibleProjects;

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

// The pie sorts wedges by year, so the selected index has to use that same order.
$: filteredYear = filterProjects.filter( (project) => {
    let years = d3.sort(pieData, (d) => d.label);
    selectedYear = selectedYearIndex > -1 ? years[selectedYearIndex]?.label : null;
    return project.year === selectedYear;
});

$: visibleProjects = (filteredYear.length > 0 ? filteredYear : filterProjects)
    .slice()
    .sort((a, b) => b.year - a.year);

</script>

<style>
input.search-bar {
    display: block;
    width: min(100%, 36rem);
    height: auto;
    margin: 0.25rem auto 1.25rem;
    font-size: 1rem;
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
        {#each visibleProjects as p (p.title)}
            <Project info={p}/>
        {/each}
    </div>
