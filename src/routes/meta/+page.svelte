<script>
    import * as d3 from "d3";
    import Pie from "$lib/Pie.svelte";
    import Scatterplot from "./Scatterplot.svelte";
    import FileLines from "./FileLines.svelte";
    import { onMount } from "svelte";
    import Scrolly from "svelte-scrolly";

    let data = [];
    let selectedCommits = [];
    let commitProgress = 50;
    let brushedSelection;
    const format = d3.format(".1~%");
    let colors = d3.scaleOrdinal(d3.schemeTableau10);

    $: hasSelection = brushedSelection || selectedCommits.length > 0;
    $: timeScale = d3.scaleTime().domain(d3.extent(commits, d => d.datetime)).range([0, 100]);
    $: commitMaxTime = timeScale.invert(commitProgress);
    $: lineMaxTime = timeScale.invert(commitProgress);
    $: d3.sort(commits, d => -d.totalLines);
    $: filteredCommits = commits.filter((commit) =>  commit.datetime < commitMaxTime);
    $: selectedLines = (hasSelection ? selectedCommits: filteredCommits).flatMap((d) => d.lines);
    $: filteredLines = selectedLines.filter( (d) => d.datetime < lineMaxTime);
    
    onMount(async() => {
        data = await d3.csv("loc.csv", row => ({
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
                value: lines,
                configurable: true,
                writable: true,
                enumerable: false
            });

            return ret;

        });

    $: fileLengths = d3.rollups(filteredLines, f => d3.max(f, v => v.totalLines), d => d.file);
    $: avgFileLength = d3.mean(fileLengths, d => d[1]);
    $: workByPeriod = d3.rollups(filteredLines, v=> v.length, d => d.datetime.toLocaleString("en", {dayPeriod: "long"}) );

    $: languageBreakdown = d3.rollup(selectedLines, v => v.length, d => d.type);   
    $: maxPeriod = d3.greatest(workByPeriod, (d) => d[1])?.[0];

</script>

<style>
    :global(body){
        max-width: min(120ch, 80vw);
    }
    
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

    .filtering{
        display: grid;
        grid-template-columns: 2.5fr 12fr;
        grid-template-rows: 1fr 1fr;
        margin-top:25px;
    }

    .slider{
        flex: 1;
    }

    .timeLabel{
        grid-column: 2;
        text-align: right;
    }
   
</style>

<div class="meta_container">
    <dl class="stats">
        <dt>COMMITS</dt>
        <dd>{filteredCommits.length}</dd>
    
        <dt>FILES</dt>
        <dd>{d3.group(filteredLines, d=> d.file).size}</dd>
    
        <dt>MAX DEPTH</dt>
        <dd>{d3.max(filteredLines, d => d.depth)}</dd>
        
        <dt>Total <abbr title="Lines of Code"> LOC</abbr></dt>
        <dd>{filteredLines.length}</dd>
    
        <dt>AVG FILE LENGTH</dt>
        <dd>{parseInt(avgFileLength)}</dd>
        
        <dt>MAX LINES</dt>
        <dd>{d3.max(filteredLines, d => d.line)}</dd>    
    </dl>
</div>

<Scrolly bind:progress={ commitProgress }>
    {#each commits as commit, index}
        <p>
            On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"} )},
            I made <a href="{commit.url}" target="_blank">{index > 0 ? 'another glorious commit' : 'my first commit'}</a>.
            I edited {commit.totalLines} lines for this commit.
            Then I looked over all I had made, and I saw that it was very good.
        </p>
    {/each}

    <svelte:fragment slot="viz">
        <h2>Commits by time of Day</h2>
        <Scatterplot commits={filteredCommits} bind:selectedCommits={selectedCommits} />
        
        {#if selectedCommits.length === 1}
            <p>{hasSelection ? selectedCommits.length : "No"} commit selected</p>
        {/if}
        
        {#if selectedCommits.length !== 1}
            <p>{hasSelection ? selectedCommits.length : "No"} commits selected</p>
        {/if}
        
        <Pie data={Array.from(languageBreakdown).map( ([language, lines]) => ({label: language, value: lines }) ) } fillColors={colors}></Pie>            
    
    </svelte:fragment>
</Scrolly>

<Scrolly bind:progress={ commitProgress } --scrolly-layout="viz-first"
throttle={1000} debounce={500}>
    {#each commits as commit, index}
        <p>
            On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"} )},
            I made <a href="{commit.url}" target="_blank">{index > 0 ? 'another glorious commit' : 'my first commit'}</a>.
            I edited {commit.totalLines} lines for this commit.
            Then I looked over all I had made, and I saw that it was very good.
        </p>
    {/each}
    <svelte:fragment slot="viz">
        <FileLines lines={filteredLines} colors={colors}/>
    </svelte:fragment>
</Scrolly>




    
    

