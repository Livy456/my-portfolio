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
    
    <script>
        // add option to change the color scheme of the page
        document.body.insertAdjacentHTML("afterbegin", 
        `<label class="color-scheme">
            Theme:
            <select>
                <option value="light dark">Automatic</option>
                <option value="light">Light</option>
                <option value="dark">Dark</option>
            </select>
        </label>
        `);

        let select = document.querySelector("select");
        if ("colorScheme" in localStorage)
        {   
            document.documentElement.style.setProperty("color-scheme", localStorage.colorScheme);
            select.value = localStorage.colorScheme;
        }
        
        select.addEventListener("input", function (event){
            
            document.documentElement.style.setProperty("color-scheme", event.target.value);
            localStorage.colorScheme = event.target.value;
        });
    </script>
    {/each}  
          
</nav>