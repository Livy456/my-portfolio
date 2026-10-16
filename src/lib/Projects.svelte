<script>
    export let info ={};
    let p;
    $: p = info; // makes p variable reactive
    
    export let hLevel = 2;
</script>

<style>
    h2, h3 {
        margin: 0;
        font-family: Garamond, Palatino, Georgia, serif;
        font-size: 1.35rem;
        font-weight: 600;
        line-height: 1.25;
    }

    dl {
        display: flex;
        flex-direction: column;
        gap: 0.35rem;
        flex: 1;
        margin: 0;
    }

    time {
        color: var(--muted, gray);
        font-size: 0.92rem;
    }

    p {
        margin: 0;
        color: var(--muted, inherit);
        font-size: 0.98rem;
    }

    .project_url {
        width: fit-content;
        margin-top: auto;
        padding-top: 0.35rem;
        color: var(--accent-strong, steelblue);
        font-weight: 600;
        text-decoration: none;
    }

    .project_url:hover,
    .project_url:focus-visible {
        text-decoration: underline;
    }
</style>

<article>
    <svelte:element this={"h" + hLevel} > {p.title}</svelte:element>

    <img src={p.image} alt={p.title}>
    <dl class="project_description">
        <time>c. {p.year}</time>
        <p>{p.description}</p>
        {#if p.url}
            <a class="project_url" href={p.url} target="_blank" rel="noreferrer">Link to the Project</a>
        {/if}
    </dl>
</article>









