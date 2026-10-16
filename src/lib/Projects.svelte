<script>
    export let info ={};
    let p;
    $: p = info; // makes p variable reactive
    
    export let hLevel = 2;
</script>

<style>
    .project_url{
        color:black;
        background-color: none;
    }

</style>

<article> 
    
    <svelte:element this={"h" + hLevel} > {p.title}</svelte:element>

    <img src={p.image} alt={p.title}>
    <dl class="project_description">
        <time style="color: gray">c. {p.year}</time>
        <p>{p.description}</p>
        {#if p.url}
            <a style="color:steelblue " class="project_url" href={p.url} target="_blank" rel="noreferrer">Link to the Project</a>
        {/if}
    </dl>
</article>









