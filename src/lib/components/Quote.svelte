<script>
    let { children, author = "Not known" } = $props();
    let quoteElement = $state();
    let status = $state("");
 
    function quoteText() {
        return quoteElement?.innerText.trim() ?? "";
    }
 
    async function copyQuote() {
        try {
            await navigator.clipboard.writeText(quoteText());
            status = "Zitat kopiert.";
        } catch {
            status = "Kopieren ist gerade nicht möglich.";
        }
    }
 
    async function share() {
        if (!navigator.share) {
            await copyQuote();
            return;
        }
 
        try {
            await navigator.share({
                title: `Zitat von ${author}`,
                text: quoteText()
            });
            status = "";
        } catch (error) {
            if (error.name !== "AbortError") {
                status = "Teilen ist gerade nicht möglich.";
            }
        }
    }
</script>
 
<svelte:head>
    <link
        href="https://fonts.googleapis.com/css2?family=Titillium+Web:ital,wght@0,300;1,300&display=swap"
        rel="stylesheet"
    />
</svelte:head>
 
<figure class="m-0 rounded-r-lg border-l-2 border-sky-600 bg-sky-50/70 p-5 sm:p-7">
    <div class="mb-4 flex items-start justify-between gap-4">
        <figcaption class="pt-1.5 text-xs font-medium tracking-wide text-sky-800">
            {author}
        </figcaption>
 
        <div class="-mr-2 flex shrink-0 items-center gap-1">
            <button
                type="button"
                onclick={copyQuote}
                class="cursor-pointer rounded px-2 py-1.5 text-xs text-sky-700 transition-colors hover:bg-sky-100 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-sky-600 motion-reduce:transition-none"
            >
                Kopieren
            </button>
 
            <span aria-hidden="true" class="h-3 w-px bg-sky-200"></span>
 
            <button
                type="button"
                onclick={share}
                class="cursor-pointer rounded px-2 py-1.5 text-xs text-sky-700 transition-colors hover:bg-sky-100 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-sky-600 motion-reduce:transition-none"
            >
                Teilen ↗
            </button>
        </div>
    </div>
 
    <blockquote
        bind:this={quoteElement}
        class="m-0 max-w-[42ch] break-words font-['Titillium_Web'] text-2xl font-light italic leading-relaxed text-slate-800 sm:text-3xl"
    >
        {@render children?.()}
    </blockquote>
 
    <p role="status" class="m-0 text-xs text-sky-700 empty:hidden [&:not(:empty)]:mt-3">
        {status}
    </p>
</figure>