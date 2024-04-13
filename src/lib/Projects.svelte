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

    <img src={p.image}>
    <dl class="project_description">
        <time>c. {p.year}</time>
        <p>{p.description}</p> 
        <a class="project_url" href={p.url} target="_blank">Link to the Project</a>
    </dl>
</article>









