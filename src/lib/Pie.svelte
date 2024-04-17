<script>
    import * as d3 from 'd3';
    export let data = [];
    export let selectedIndex = -1;
    export let fillColors = d3.scaleOrdinal(d3.schemeTableau10);
    
    // returns a function that uses index to produce a color
    let sliceGenerator = d3.pie().value(d => d.value).sort(null);
    let pieData;
    let transitionDuration = 3000;
    let oldData = [];
    let wedges = {};
    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);

    $:{
        let arcData;
        let arcs;
        oldData = pieData;
        // pieData = data.map(d => ({...d}) );
        pieData = d3.sort(data, d => d.label);
        arcData = sliceGenerator(pieData);
        arcs = arcData.map(d => arcGenerator(d));
        pieData = pieData.map( (d, i) =>  ({...d, ...arcData[i], arc: arcs[i]}) );
        transitionArcs();
    }    

    function toggleWedge(index, event){
        if(selectedIndex === index)
        {
            selectedIndex = -1;
        }
        
        else if(!event.key || event.key === "Enter")
        {
            selectedIndex = index;
        }
    }

    function transitionArcs() {
        let wedgeElements = Object.values(wedges);
        
        d3.selectAll(wedgeElements).transition("arc")
            .duration(transitionDuration)
            .styleTween("d", function (_, index ) {
                let wedge = this;
                let label = Object.keys(wedges)[index];
                let transition = transitionArc(wedge, label);

                // returns interpolator if it is defined
                return transition?.interpolator;
            }).ease(d3.easeLinear);
    }

    function getEmptyArc (label, data = pieData){
        // Union of old and new labels in the order they appear
        let labels = d3.sort( new Set([...oldData, ...pieData].map(d=> d.label) ) );
        let labelIndex = labels.indexOf(label);
        let sibling;

        for(let i = labelIndex - 1; i >= 0; i--){
            sibling = data.find(d => d.label === labels[i]);
            if (sibling){
                break;
            }
        }   

        let angle = sibling?.endAngle ?? 0;
        return {startAngle: angle, endAngle: angle};
    } 

    function arc (wedge){
        // Calculations that will only be done once per element go here
        return{
            duration: transitionDuration,
            css: (t, u) => {
                // t is a number between 0 and 1 that represents the transition progress; u is 1 - t
                let wedge = this;
                // let label = Object.keys(wedges)[index]; 
                let transition = transitionArc(wedge);
                return `{transition.interpolator(transition.type === "out" ? u : t)}`;
            }
        }
    }

    function transitionArc (wedge, label){
        label ??= Object.entries(wedges).find( ([label, w]) => w === wedge )[0];

        let d = pieData.find(d => d.label === label);
        let d_old = oldData.find(d => d.label === label);

        if (sameArc(d_old, d) )
        {
            return null;
        }

        let from = d_old ? {...d_old} : getEmptyArc(label, oldData);
        let to = d ? {...d} : getEmptyArc(label, pieData);
        let angleInterpolator = d3.interpolate(from, to);
        let type = d ? d_old ? "update" : "in" : "out";

        // Calculations that will only be done once per element go here
        let interpolator = t => `path("${ arcGenerator(angleInterpolator(t)) }")`;

        // return interpolator;

        return {d, d_old, from , to, interpolator};

    }

    function sameArc(d_old, d){
        // checks if either arc is undefined or null
        if(d_old === null || d_old === undefined || 
           d === null || d === undefined){
            return true;
        }

        // checks if both arcs have same start and end angle
        else if(d.startAngle == d.endAngle && d_old.endAngle == d.endAngle){
            return true;
        }

        // defaults to arcs not being the same
        return false;
    }   

</script>

<style>
    
    svg{
        max-width: 20em;
        margin-block: 2em;
        /* Doesn't clip the shape even if it'd outside of viewbox */
        overflow: visible;
        padding-bottom:25px;
    }

    svg:has(path:hover, path:focus-visible)
    {
        path:not(:hover, :focus-visible){
            opacity: 50%;
        }
    }

    path{
        transition: 300ms;
        transition-property: transform, opacity, fill;
        outline: none;
        cursor: pointer;
        fill-opacity: 75%;
        --angle: calc(var(--end-angle) - var(--start-angle));
        --mid-angle: calc(var(--start-angle) + var(--angle) / 2);
        transform: rotate(var(--mid-angle)) translateY(0) rotate(calc(-1 * var(--mid-angle)));

        &.selected{
            transform: rotate(var(--mid-angle)) translateY(-6px) scale(1.1)
            rotate(calc(-1 * var(--mid-angle)));
        }
    }

    .container{
        align-items: center;
        flex: 1;
        display: flex;
        gap: 60ch;
    }

    span.swatch{
        display: inline-block;
        aspect-ratio: 1 / 1;
        background-color: var(--color);
        width: 15px;
        height: 15px;
        border-radius: 50%;
    }

    ul.legend{
        /* display: grid; */
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        border-color: rgb(193, 193, 193);
        border-width: 3px;
        border-style: solid;
        padding-left: 30px;
        padding-right: 30px;
        padding-top: 10px;
        padding-bottom: 10px;
    }

    li{
        font-palette: black;
        list-style: none;
    }

    .selected {
        --color: oklch(60% 45% 0) !important;

        &:is(path) {
            fill: var(--color);
        }
    }

</style>

<div class="container">

    <svg viewBox= "-190 -55 100 100">
        {#each pieData as d, i (d.label)}
            <path 
            bind:this={ wedges[d.label] }
            d={d.arc} 
            style="
            --start-angle: { d.startAngle }rad;
            --end-angle: { d.endAngle }rad;"
            transition:arc
            fill= {fillColors(d.id ?? d.label) } 
            tabindex="0"
            aria-label="Pie Chart"
            role= "button"
            class:selected={selectedIndex === i} 
            on:click={e => toggleWedge(i, e)} 
            on:keyup={e => toggleWedge(i, e)}/>
        {/each}  
    </svg>
    <ul class="legend">
        {#each pieData  as d, index}
            <li style="--color: { fillColors(d.label) }">
                <span class="swatch" class:selected={selectedIndex === index}></span>
                {d.label} <em> ({d.value})</em>
            
            </li>
            
        {/each}
    </ul>
</div>
    