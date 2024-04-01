<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    let data = [];
    let commits = [];

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
    
</style>

<!-- <p>Total lines of code: {data.length}</p> -->
<!-- <p>{JSON.stringify(commits, null, "\t")}</p> -->
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

<!-- <dl class="date_summary">
    <dt>DAY OF THE WEEK THAT MOST WORK IS DONE</dt>
    <dd>{maxPeriod}</dd>
</dl> -->