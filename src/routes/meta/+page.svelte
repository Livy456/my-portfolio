<script>
    import * as d3 from "d3";
    import Pie from "$lib/Pie.svelte";
    import CommitScatterplot from "../meta/Scatterplot.svelte";
    import { onMount } from "svelte";
    import {
        computePosition,
        autoPlacement,
        offset
    } from '@floating-ui/dom';

    let data = [];
    let commits = [];
    // let svg;
    let selectedCommits = [];
    let languageBreakdown;
    let languageBreakdownArray;
    let commitProgress = 100;
    let commitMaxTime; 
    let timeScale = d3.scaleTime();
    let filteredCommits;
    let brushedSelection;
    let filteredCommitsLines;
    let hasSelection = undefined;
    const format = d3.format(".1~%");

    // $: selectedCommits = brushedSelection ? commits.filter(isCommitSelected) : []; 
    // $: hoveredCommit = filteredCommits[hoveredIndex]?? {};   
    $: hasSelection = brushedSelection || selectedCommits.length > 0;
    // $: selectedLines = (hasSelection ? selectedCommits: commits).flatMap(d => d.lines);
    // $: selectedLines = hasSelection ? data.filter((d) => selectedCommits.map(commit => commit.id).includes(d.commit)) : data;
    // $: filteredCommitsLines = data.filter ((d) => d.datetime < commitMaxTime);
    // $: filteredCommits = commits.filter((commit) =>  commit.datetime < commitMaxTime);
    $: timeScale = timeScale.domain(d3.extent(commits, d => d.datetime)).range([0, 100]);
    $: commitMaxTime = timeScale.invert(commitProgress);
    $: commits = d3.sort(commits, d => -d.totalLines);
    $: filteredCommits = commits.filter((commit) =>  commit.datetime < commitMaxTime);
    $: filteredCommitsLines = data.filter ((d) => d.datetime < commitMaxTime);

    onMount(async() => {
        data = await d3.csv("loc.csv", row=> ({
            ...row,
            type: String(row.type),
            line: Number(row.line),
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));
    });

    $: commits = d3.groups(data, d => d.commit).map( ([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/Livy456/my-portfolio" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length
            };

            Object.defineProperty(ret, "lines", {
                values: lines,
                configurable: true,
                writable: true,
                enumerable: false
            });

            return ret;

        });
    
    $: fileLengths = d3.rollups(filteredCommitsLines, v => d3.max(v, v => v.line), d => d.file);
    $: avgFileLength = d3.mean(fileLengths, f => f[1]);
    $: workByPeriod = d3.rollups(filteredCommitsLines, v=> v.length, d => d.datetime.toLocaleString("en", {dayPeriod: "long"}) );
    
    // BUG WITH NOT UPDATING THE PIE CHART WHEN SELECTING COMMITS OR CLICKING ON A SINGLE COMMIT 
    $: languageBreakdown = d3.rollups(filteredCommitsLines, v=> v.length, d => d.type);    
    
    $: languageBreakdownArray = Array.from(languageBreakdown).map( ([language, lines]) => ({label: language, value:lines}) );
    $: maxPeriod = d3.greatest(workByPeriod, (d) => d[1])?.[0];
    
</script>

<style>
    h2.meta{
        font-size: 50px;
        font-family: 'Segoe UI';
    }

    dl.stats{
        display: grid;
        grid-template-columns: auto;
        grid-template-rows: auto;
        font-family: 'Segoe UI';
        padding-top: 0px;
        margin-top: 0px;

        dt{
            grid-row: 1;
            font-size: 20px;
            color: grey;
        }

        dd{
            grid-row: 2;
            font-size: 35px;
            text-align: left;
            padding: 0px;
            margin: 0px;
        }
    }

    dl.date_summary{
        display: grid;
        grid-template-columns: auto;
        grid-template-rows: auto;
        font-family: 'Segoe UI';
    }

    .meta_legend{
        display: grid;


        dd{
            grid-row: 1;
            margin: 0px;
            font-size: 15px;
            text-transform: uppercase;
            color: rgb(99, 99, 99);
            font-family: 'Segoe UI';
        }

        dt{
            grid-row: 2;
        }
    }
    
</style>
<label class="filtering">
    <p>Show commits until:</p> 
    <input class="slider" type="range" min="1" max="100" bind:value={commitProgress}>
    <time class="timeLabel">{commitMaxTime.toLocaleString("en", {dateStyle: "long", timeStyle: "short"})}</time> 
</label>

<div class="meta_container">
    <dl class="stats">
        <dt>COMMITS</dt>
        <dd>{filteredCommits.length}</dd>
    
        <dt>FILES</dt>
        <dd>{d3.group(filteredCommitsLines, d=> d.file).size}</dd>
    
        <dt>MAX DEPTH</dt>
        <dd>{d3.max(filteredCommitsLines, d => d.depth)}</dd>
        
        <dt>Total <abbr title="Lines of Code"> LOC</abbr></dt>
        <dd>{filteredCommitsLines.length}</dd>
    
        <!-- <dt>AVG FILE DEPTH</dt>
        <dd>{parseInt(d3.mean(data, d => d.depth))}</dd> -->
    
        <dt>AVG FILE LENGTH</dt>
        <dd>{parseInt(avgFileLength)}</dd>
    
        <dt>MAX LINES</dt>
        <dd>{d3.max(filteredCommitsLines, d => d.line)}</dd>    
    </dl>
    
    <h2 style="margin-top: 3rem">Commits by time of Day</h2>
    <CommitScatterplot commits={filteredCommits} bind:selectedCommits={selectedCommits} />
    
    {#if selectedCommits.length === 1}
        <p>{hasSelection ? selectedCommits.length : "No"} commit selected</p>
    {/if}
    
    {#if selectedCommits.length !== 1}
        <p>{hasSelection ? selectedCommits.length : "No"} commits selected</p>
    {/if}

    <dl class="meta_legend">
        <!-- {console.log("selectedCommits: ", selectedCommits)} -->
        <!-- {console.log("languageBreakdownArray: ", languageBreakdownArray)} -->
        <Pie data={languageBreakdownArray}></Pie>
    </dl>
</div>

