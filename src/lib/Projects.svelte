<script>
    export let info ={};
    let p;
    $: p = info; // makes p variable reactive
    
    export let hLevel = 2;
</script>

<article> 
    
    <svelte:element this={"h" + hLevel} > {p.title}</svelte:element>

    <!-- <img src={p.image} /> -->
    <img src={p.image}>
    <p>{p.description}</p>
    <time>c. {p.year}</time>
</article>






