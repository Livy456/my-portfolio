<svelte:head>
    <title>Contact Me</title>
</svelte:head>

<h1>Contact</h1>
    <form action="mailto:Marvelfanatic369@gmail.com" method="POST" enctype="text/plain">
        <label>Sender's Email: <input name="email" type="email"></label>
        <label>Subject Line: <input name="subject" type="text"></label>
        <label>Message: <textarea name="body"></textarea></label>
        <button>Submit</button>
    </form>