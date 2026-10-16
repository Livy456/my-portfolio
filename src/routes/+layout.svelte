<script>
    import Navbar from "../lib/Navbar.svelte";

    let localStorage = globalThis.localStorage ?? {};
    let colorScheme = "light dark";
    $: localStorage.colorScheme = colorScheme;
    // console.log??(localStorage.colorScheme);

    // prevents color scheme from going to 
    // default settings when page is reloaded
    if (localStorage.colorScheme) {
        colorScheme = localStorage.colorScheme;
    }

    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);

    // let profileDate = fetch("https://api.github.com/users/livy456");

</script>
<style>
label.color-scheme {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.5rem;
    max-width: 100%;
    margin: 0 0 0.85rem;
    padding: 0;
    color: inherit;
    font-size: 0.9rem;
}

label.color-scheme select {
    max-width: 100%;
    padding: 0.2rem 0.45rem;
}
</style>
<Navbar></Navbar>

<label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

<slot />