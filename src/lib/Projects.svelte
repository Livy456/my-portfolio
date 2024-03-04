<script>
    export let info ={};
    let p = info;
    
    export let hLevel = 2;
</script>

<article>
    <!-- {#if hLevel === 1}
        <h1>{p.title}</h1>
    
    {:else if hLevel === 2}
        <h2>{p.title}</h2>

    {:else if hLevel === 3}
        <h3>{p.title}</h3>
    
    {:else if hLevel === 4}
        <h4>{p.title}</h4>

    {:else if hLevel === 5}
        <h5>{p.title}</h5>

    {:else if hLevel === 6}
        <h6>{p.title}</h6>
    {/if} -->


    <svelte:element this={"h" + hLevel} > {p.title}</svelte:element>

    <!-- <img src={p.image} /> -->
    <img src={p.image}>
    <p>{p.description}</p>
</article>

