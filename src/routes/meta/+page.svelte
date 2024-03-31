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

        console.log(commits);
    });



    
</script>

<p>Total lines of code: {data.length}</p>
<!-- <p>{JSON.stringify(commits, null, "\t")}</p> -->