<script lang="ts">
    import '../pico.css';
    let inputUrl = '';
    let shortenedUrl = '';
    let error = '';

    async function handleSubmit() {
        try {
            const formData = new FormData();
            formData.append('url', inputUrl);

            const response = await fetch('/api', {
                method: 'POST',
                body: formData,
            });

            if (!response.ok) {
                throw new Error(await response.text());
            }

            const { result_url } = await response.json();
            shortenedUrl = result_url;
            error = '';
        } catch (err) {
            error = (err as Error).message;
            shortenedUrl = '';
        }
    }
</script>

<header>
    <h1>URL Shortener</h1>
</header>
<main>
    <form onsubmit={handleSubmit}>
        <input
            type="url"
            bind:value={inputUrl}
            placeholder="Enter URL to shorten"
        />
        <button type="submit">Shorten It!</button>
    </form>

    {#if error}
        <p class="error">{error}</p>
    {/if}
    {#if shortenedUrl}
        <div class="shortened">
            <p>
                Shortened URL: <a href={shortenedUrl} target="_blank"
                    >{shortenedUrl}</a
                >
            </p>
            <button
                onclick={() => {
                    navigator.clipboard.writeText(shortenedUrl);
                }}>Copy</button
            >
        </div>
    {/if}
</main>

<style>
    p {
        margin: 0;
    }

    h1 {
        text-align: center;
        margin-top: 5rem;
    }

    main {
        max-width: 768px;
        margin: 0 auto;
    }
    form,
    .shortened {
        display: flex;
        justify-content: space-between;
        align-items: center;
        gap: 1rem;
    }

    button {
        max-width: 200px;
        cursor: pointer;
    }

    .shortened {
        border: 1px solid var(--pico-secondary-border);
        padding: 1rem;
        border-radius: 10px;
        background-color: #181c25;
    }
</style>
