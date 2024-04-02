<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    let data = [];
    let commits = [];
    let width = 900, height = 450; // changed the height of the graph from 600 to 450
    let yScale = d3.scaleLinear();
    let xScale = d3.scaleTime();
    let xAxis, yAxis;
    let yAxisGridlines;

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

    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
        d3.select(yAxisGridlines).call(d3.axisLeft(yScale).tickFormat("").tickSize(-usableArea.width));
    }

    onMount(async() => {
        data = await d3.csv("loc.csv", row=> ({
            ...row,
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

        yScale = yScale.domain([0, 24]).range([usableArea.bottom, usableArea.top]); // might need to switch values currently domain = [0, height], range = [0, 24] 
        xScale = xScale.domain( [ d3.min(commits, d => d.datetime), d3.max(commits, d => d.datetime) ] ).range( [usableArea.left, usableArea.right] );
        // NOT SURE HOW TO GET THE scale.nice() scaling
        // ALSO ERROR IS SAYING THAT COMMITS.DATETIME IS NOT ITERABLE
        
        // scaleTime(commits.datetime, d3.extent(commits.datetime))
        // .domain(commits.datetime).nice();
        // MAKING XSCALE MIGHT BE CAUSE OF FUTURE ERRORS
        // d3.scaleTime(data.datetime, d3.extent(data.datetime));

    });

    $: fileLengths = d3.rollups(data, v => d3.max(v, v => v.line), d => d.file);
    $: avgFileLength = d3.mean(fileLengths, f => f[1]);
    $: workByPeriod = d3.rollups(data, v=> v.length, d => d.datetime.toLocaleString("en", {dayPeriod: "short"}) );
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
    }

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

    dl.date_summary{
        display: grid;
        grid-template-columns: auto;
        grid-template-rows: auto;
        font-family: 'Segoe UI';
    }

    svg{
        overflow: visible;
        margin:25px;
    }
    
</style>
<h2 class="meta">Summary</h2>

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

<h2 style="margin-top: 3rem">Commits by time of Day</h2>

<svg viewBox="0 0 {width} {height}">
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
    <g transform="translate({usableArea.top})" bind:this={yAxis}/>
    
    <g class="dots">
    {#each commits as commit, index}
        <circle 
            cx={ xScale(commit.datetime) }
            cy={ yScale(commit.hourFrac) }
            r="5"
            fill="red"
        />
    {/each}
    </g>

</svg>

<!-- <dl class="date_summary">
    <dt>DAY OF THE WEEK THAT MOST WORK IS DONE</dt>
    <dd>{maxPeriod}</dd>
</dl> -->