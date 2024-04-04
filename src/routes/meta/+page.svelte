<script>
    import * as d3 from "d3";
    import Pie from "$lib/Pie.svelte";
    import { onMount } from "svelte";
    import {
        computePosition,
        autoPlacement,
        offset
    } from '@floating-ui/dom';

    let data = [];
    let commits = [];
    let width = 900, height = 475; // changed the height of the graph from 600 to 450
    let yScale = d3.scaleLinear();
    let xScale = d3.scaleTime();
    let rScale = d3.scaleSqrt();
    let xAxis, yAxis;
    let yAxisGridlines;
    let hoveredIndex = -1;
    $: hoveredCommit = commits[hoveredIndex]?? {};
    let tooltipPosition = {x:0, y:0};
    let commitTooltip;
    let svg;
    let brushedSelection;
    let selectedCommits;
    let hasSelection;
    let selectedLines;
    let languageBreakdown;
    let languageBreakdownArray;

    // let brushedSelection;
    $: selectedCommits = brushedSelection ? commits.filter(isCommitSelected) : [];    
    $: hasSelection = brushedSelection && selectedCommits.length > 0;
    // $: selectedLines = (hasSelection ? selectedCommits: commits).flatMap(d => d.lines);
    $: selectedLines = hasSelection ? data.filter((d) => selectedCommits.map(commit => commit.id).includes(d.commit)) : data;

    const format = d3.format(".1~%");
        
    // defining axes
    let margin = {top: 10, right: 10, bottom: 30, left:20};
    let usableArea = {
        top: margin.top,
        bottom: height - margin.bottom,
        left: margin.left,
        right: width - margin.right
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;

    // creating axis on page
    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
        d3.select(yAxisGridlines).call(d3.axisLeft(yScale).tickFormat("").tickSize(-usableArea.width));
    }

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

        commits = d3.groups(data, d => d.commit).map( ([commit, lines]) => {
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

        commits = d3.sort(commits, d => -d.totalLines);

        yScale = yScale.domain([0, 24]).range([usableArea.bottom, usableArea.top]); // might need to switch values currently domain = [0, height], range = [0, 24] 
        xScale = xScale.domain(d3.extent(commits, d => d.datetime)).range( [usableArea.left, usableArea.right] ).nice();
        rScale = rScale.domain(d3.extent(commits, d => d.totalLines)).range([5, 30]);
        
    });

    $: fileLengths = d3.rollups(data, v => d3.max(v, v => v.line), d => d.file);
    $: avgFileLength = d3.mean(fileLengths, f => f[1]);
    $: workByPeriod = d3.rollups(data, v=> v.length, d => d.datetime.toLocaleString("en", {dayPeriod: "short"}) );
    $: languageBreakdown = d3.rollups(selectedLines, v=> v.length, d => d.type);    
    $: languageBreakdownArray = Array.from(languageBreakdown).map( ([language, lines]) => ({label: language, value:lines}) );

    $: {
        console.log(languageBreakdown)
    }
    $: maxPeriod = d3.greatest(workByPeriod, (d) => d[1])?.[0];
    $: {
        d3.select(svg).call(d3.brush().on("start brush end", brushed));
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    async function dotInteraction (index, evt){
        let hoveredDot = evt.target;
        
        if (evt.type === "mouseenter" || evt.type === "focus")
        {
            hoveredIndex = index;
            // tooltipPosition = {x:evt.x, y:evt.y};
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed",
                middleware: [
                    offset(20),
                    autoPlacement()
                ],
            });
        }

        else if (evt.type === "mouseleave" || evt.type === "blur")
        {
            hoveredIndex = -1;
        }
    }

    function brushed (evt)
    {
        brushedSelection = evt.selection;
    }

    function isCommitSelected(commit)
    {
        if(!brushedSelection)
        {
            return false;
        }

        // get points of brushed selection box
        let top_left = {x: brushedSelection[0][0], y: brushedSelection[0][1]};
        let bottom_right = {x: brushedSelection[1][0], y: brushedSelection[1][1]};

        // gets coordinate of commit data point
        let commit_x_coord = xScale(commit.datetime);
        let commit_y_coord = yScale(commit.hourFrac)

        // checks if commit data point is within selected brush region
        if(commit_x_coord >= top_left.x && commit_x_coord <= bottom_right.x && 
           commit_y_coord >= top_left.y && commit_y_coord <= bottom_right.y)
        {
            return true;
        }

        return false;
    }
    
</script>

<style>

@keyframes marching-ants {
        to{
            stroke-dashoffset: -8;
        }
    }

    svg :global(.selection){
        /* overflow: visible;
        margin:50px; */
        fill-opacity: 10%;
        stroke: black;
        stroke-opacity: 70%;
        stroke-dasharray: 5 3;
        animation: marching-ants 2s linear infinite;
    }

    svg{
        margin:50px;
    }

    dl.info
    {
        display: grid;
        grid-template-columns: auto 1fr;
        background-color: oklch(100% 0% 0 / 80%);
        box-shadow: 5px 5px 5px lightslategrey;
        border-radius: 5%;
        backdrop-filter: blur(10px);
        padding:10px;

        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within){
            opacity: 0;
            visibility: hidden;
        }
    }

    .tooltip{
        position: fixed;
        margin:15px;
        /* top: 1em; */
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

    .gridlines{
        stroke-opacity: 0.2;
    }
    
    .dots{

        circle.selected{
            fill: rgb(228, 24, 24)
        }

        circle{
            transition: 200ms;
            transform-origin: center;
            transform-box: fill-box;
            /* fill-opacity: 100%;  */
            /* HOW DO YOU CHANGE THE OPACITY OF BIGGER DOTS */

            &:hover
            {
                transform: scale(1.5);
                
            }
        }
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
<h2 class="meta">Summary</h2>

<h2 style="margin-top: 3rem">Commits by time of Day</h2>

<div class="meta_container">
    <dl class="stats">
        <dt>COMMITS</dt>
        <dd>{commits.length}</dd>
    
        <dt>FILES</dt>
        <dd>{d3.group(data, d=> d.file).size}</dd>
    
        <dt>MAX DEPTH</dt>
        <dd>{d3.max(data, d => d.depth)}</dd>
        
        <dt>Total <abbr title="Lines of Code"> LOC</abbr></dt>
        <dd>{data.length}</dd>
    
        <!-- <dt>AVG FILE DEPTH</dt>
        <dd>{parseInt(d3.mean(data, d => d.depth))}</dd> -->
    
        <dt>AVG FILE LENGTH</dt>
        <dd>{parseInt(avgFileLength)}</dd>
    
        <dt>MAX LINES</dt>
        <dd>{d3.max(data, d => d.line)}</dd>    
    </dl>
    
    <dl id="commit-tooltip" class="info tooltip" 
        hidden={hoveredIndex === -1}
        bind:this={commitTooltip}
        style="top:{tooltipPosition.y}px; left:{tooltipPosition.x}px">
        <dt>Commit</dt>
        <dd> <a href="{ hoveredCommit.url}" target="_blank"> { hoveredCommit.id }</a> </dd>
    
        <dt>Date</dt>
        <dd>{ hoveredCommit.datetime?.toLocaleString("en", {date: "full"}) }</dd>
    
        <dt>Time</dt>
        <dd>{ hoveredCommit.time }</dd>
    
        <dt>Author</dt>
        <dd>{ hoveredCommit.author }</dd>
    
        <dt>Lines</dt>
        <dd>{ hoveredCommit.totalLines }</dd>

        <!-- <dt>Language</dt>
        <dd>{ hoveredCommit.language }</dd> -->
    </dl>
    <svg viewBox="0 0 {width} {height}" bind:this={svg}>
        <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
        <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
        <g transform="translate({usableArea.top})" bind:this={yAxis}/>
        
        <g class="dots">
        {#each commits as commit, index}
            <circle 
                class:selected={isCommitSelected(commit)}
                cx={ xScale(commit.datetime) }
                cy={ yScale(commit.hourFrac) }
                r= { rScale(commit.totalLines) }
                fill="#B19CD9"
                fill-opacity= {(105 - rScale(commit.totalLines))/100}
                on:mouseenter= {evt=> dotInteraction(index, evt)}
                on:mouseleave={evt => dotInteraction(index, evt)}
                tabindex="0"
                aria-describedby="commit-tooltip"
                role="tooltip"
                aria-haspopup="true"
                on:focus={evt=> dotInteraction(index, evt)}
                on:blur={evt=> dotInteraction(index, evt)}
            />
            <!-- FUTURE EXPLORATION- MAKE A FUNCTION THAT CHANGES THE COLOR BASED ON TIME OF DAY 
                    I.E. MORNING IS ORANGE AND NIGHT IS BLUE -->
        {/each}
        </g>
    </svg>

    <p>{hasSelection ? selectedCommits.length : "No"} commits selected</p>
    <dl class="meta_legend">
        <Pie data={languageBreakdownArray}></Pie>
    </dl>
    
    
</div>

