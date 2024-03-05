<script>
    import { page } from '$app/stores';
    
    let pages = [
        {url: "./", title:"Home"},
        {url: "./projects", title:"Projects"},
        {url: "./contact", title:"Contact"},
        {url: "https://livy456.github.io/my-portfolio/", title:"Portfolio"},
        {url: "./resume", title:"Resume"}
    ]

</script>
<!-- { JSON.stringify($page) }       -->

<nav>

    {#each pages as p}
        <a href={ p.url } class:current={ "." + $page.route.id === p.url } target= { p.url.startsWith("http") ? "._blank" : null }>{ p.title }</a>
    {/each}  
          
</nav>