<script>
    import * as d3 from "d3";
    import {
        computePosition,
        autoPlacement,
        offset
    } from '@floating-ui/dom';
    export let commits = [];
    export let selectedCommits = [];

    let width = 900, height = 475; // changed the height of the graph from 600 to 450
    // let yScale = d3.scaleLinear();
    // let xScale = d3.scaleTime();
    // let rScale = d3.scaleSqrt();
    let xAxis, yAxis;
    let yAxisGridlines;
    let hoveredIndex = -1;
    let tooltipPosition = {x:0, y:0};
    let commitTooltip;
    let svg;
    let brushedSelection;
    // let hasSelection = undefined;
    // let hoveredCommit;
    const format = d3.format(".1~%");

    $: hoveredCommit = commits[hoveredIndex]?? {};   
    $: hasSelection = brushedSelection || selectedCommits.length > 0;
    
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

    $: yScale = d3.scaleLinear().domain([0, 24]).range([usableArea.bottom, usableArea.top]); // might need to switch values currently domain = [0, height], range = [0, 24] 
    $: xScale = d3.scaleTime().domain(d3.extent(commits, d => d.datetime)).range( [usableArea.left, usableArea.right] ).nice();
    $: rScale = d3.scaleSqrt().domain(d3.extent(commits, d => d.totalLines)).range([5, 30]);
    
    $: {
        d3.select(svg).call(d3.brush().on("start brush end", brushed));
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    async function dotInteraction (index, evt){
        let hoveredDot = evt.target;
        
        if (evt.type === "mouseenter" || evt.type === "focus")
        {
            hoveredIndex = index;
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
        
        else if ( (evt.type === "click") || (evt.type === "brush") ||
                ( (evt.type === "keyup") && (evt.key === "Enter") ) )
        {
            // changes selected commits to be just the individual commit clicked on
            selectedCommits = [ commits[index] ];
        }
    }

    function brushed (evt)
    {
        let brushedSelection = evt.selection;
        // console.log("evt", evt);

        selectedCommits = !brushedSelection ? [] : commits.filter(commit => {
            let top_left = {x: brushedSelection[0][0], y: brushedSelection[0][1]};
            let bottom_right = {x: brushedSelection[1][0], y: brushedSelection[1][1]};

            let commit_x_coord = xScale(commit.datetime);
            let commit_y_coord = yScale(commit.hourFrac);

            // console.log("commit x: ", commit_x_coord, ", commit y: ", commit_y_coord);
            // console.log("brushed top left: ", top_left);
            // console.log("brushed bottom right: ", bottom_right);

            if ((commit_x_coord >= top_left.x) && (commit_y_coord >= top_left.y) && 
                (commit_x_coord <= bottom_right.x) && (commit_y_coord <= bottom_right.y))
            {
                console.log("I have been selected, inside brush function!!");
                hasSelection = true;
                return true;
            }
            
            return false;
        });

    }

    function isCommitSelected(commit)
    {
        // console.log("is commit selected function: ", selectedCommits);
        return selectedCommits.includes(commit);
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

            &:hover
            {
                transform: scale(1.5);
                
            }

            @starting-style{
                r:0;
            }

            transition: all 200ms,
                        r calc(var(--r) * 100ms);
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

<div class="meta_container">
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

    </dl>

    <svg viewBox="0 0 {width} {height}" bind:this={svg}>
        <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
        <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
        <g transform="translate({usableArea.top})" bind:this={yAxis}/>
        <g class="dots">
        {#each commits as commit, index (commit.id)}
            <circle 
                style="--r: {rScale(commit.totalLines)} "
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
                on:click={evt=> dotInteraction(index, evt)}
                on:keyup={evt=> dotInteraction(index, evt)}
            />

        {/each}
        </g>
    </svg>
</div>