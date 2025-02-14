<script lang="ts">
    let inputUrl = '';
    let error = '';

    let copyFeedback = false;

    interface UrlItem {
        originalUrl: string;
        shortenedUrl: string;
        copyFeedback: boolean;
    }
    let urlList: UrlItem[] = [];

    function copyToClipboard(index: number) {
        navigator.clipboard.writeText(urlList[index].shortenedUrl);
        urlList[index].copyFeedback = true;
        urlList = [...urlList];
        setTimeout(() => {
            urlList[index].copyFeedback = false;
            urlList = [...urlList];
        }, 2000);
    }

    async function handleSubmit() {
        try {
            const urlToShorten =
                inputUrl.startsWith('http://') ||
                inputUrl.startsWith('https://')
                    ? inputUrl
                    : `https://${inputUrl}`;

            const formData = new FormData();
            formData.append('url', urlToShorten);

            const response = await fetch('/api', {
                method: 'POST',
                body: formData,
            });

            if (!response.ok) {
                throw new Error(await response.text());
            }

            const { result_url } = await response.json();
            urlList = [
                ...urlList,
                {
                    originalUrl: urlToShorten,
                    shortenedUrl: result_url,
                    copyFeedback: false,
                },
            ];
            inputUrl = '';
            error = '';
        } catch (err) {
            error = (err as Error).message;
        }
    }
</script>

<svelte:head>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
        href="https://fonts.googleapis.com/css2?family=Righteous&display=swap"
        rel="stylesheet"
    />
    <link
        rel="stylesheet"
        href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.blue.min.css"
    />
</svelte:head>

<header>
    <h1>URL <br />Shortener</h1>
</header>
<main>
    <form onsubmit={handleSubmit}>
        <input
            type="text"
            bind:value={inputUrl}
            placeholder="Enter URL to shorten"
        />
        <button type="submit">Shorten It!</button>
    </form>

    {#if error}
        <p class="error">Invalid url!</p>
    {/if}
    <div class="url-list">
        {#each urlList as { originalUrl, shortenedUrl, copyFeedback }, index}
            <div class="shortened">
                <div class="url-info">
                    <p class="original-url">
                        <a
                            class="original-url no-underline"
                            href={originalUrl}
                            target="_blank">{originalUrl}</a
                        >
                    </p>
                    <p>
                        Shortened: <a href={shortenedUrl} target="_blank"
                            >{shortenedUrl}</a
                        >
                    </p>
                </div>
                <button
                    class:copied={copyFeedback}
                    onclick={() => copyToClipboard(index)}
                >
                    {copyFeedback ? 'Copied!' : 'Copy'}
                </button>
            </div>
        {/each}
    </div>
</main>
<footer>
    <p>
        created by <a
            href="https://github.com/anrelps"
            target="_blank"
            class="no-underline"
            >anrelps<svg
                viewBox="0 0 512 512"
                xmlns="http://www.w3.org/2000/svg"
                ><path
                    d="m256 0c-140.699219 0-256 116.300781-256 257 0 139.882812 114.25 255 256 255 141.574219 0 256-114.945312 256-255 0-140.699219-115.300781-257-256-257zm45 477.5c-14.398438 3-29.699219 4.5-45 4.5s-30.601562-1.5-45-4.5v-70.199219c0-16.800781 4.5-22.800781 10.5-30.902343 3.054688-3.492188 4.898438-6.625 18.597656-27.296876l-23.097656-3.601562c-59.402344-8.699219-82.800781-39.601562-92.101562-63.601562-12-32.097657-5.699219-72.300782 15.902343-97.796876 3.300781-3.902343 6-10.503906 3.601563-17.402343-4.503906-13.800781-3.902344-35.699219-.902344-44.101563 15.90625 2.273438 32.261719 13.667969 45.902344 21.902344 6.285156 3.667969 9.582031 2.699219 12.597656 3 10.960938-2.28125 28.058594-7.800781 54.300781-7.800781 16.199219 0 33.300781 2.398437 50.101563 7.199219 3.003906-.070313 7.832031 2.484374 16.199218-2.398438 14.257813-8.6875 30.058594-19.691406 45.898438-21.902344 3 8.402344 3.601562 30.300782-.898438 44.101563-2.402343 6.898437.296876 13.5 3.601563 17.402343 21.597656 25.5 27.898437 65.699219 15.898437 97.796876-9.300781 24-32.699218 54.902343-92.101562 63.601562l-23.097656 3.601562c14.160156 21.367188 15.652344 23.929688 18.601562 27.296876 5.996094 8.101562 10.496094 14.101562 10.496094 30.902343zm30-8.699219v-61.5c0-17.101562-3.601562-28.5-8.402344-36.902343 45.601563-12.296876 78.003906-39.300782 92.402344-78 15.300781-40.796876 8.402344-89.398438-17.101562-123 4.503906-20.097657 4.503906-52.199219-6.296876-67.199219-4.800781-6.597657-11.402343-10.199219-19.800781-10.199219-.300781 0-.300781 0-.300781 0-23.261719 1.257812-41.570312 12.972656-61.199219 24.898438-18-4.800782-36.300781-7.199219-54.601562-7.199219-18.597657 0-37.199219 2.699219-53.695313 7.199219-20.664062-12.460938-38.796875-23.671876-62.703125-24.898438-7.5 0-14.101562 3.601562-18.902343 10.199219-10.796876 15-10.796876 47.101562-6.296876 67.199219-25.503906 33.601562-32.402343 82.5-17.101562 123 14.398438 38.699218 46.800781 65.703124 92.402344 78-3.722656 6.511718-6.667969 14.914062-7.828125 26.285156-9.210938 3.175781-17.199219 4.210937-24.628907 2.027344-7.835937-2.316407-13.941406-7.546876-19.246093-16.46875-11.914063-20.015626-32.207031-36.355469-55.3125-34.230469l2.636719 29.882812c10.699218-.980469 21.347656 10.339844 26.878906 19.671875 9.125 15.367188 21.417968 25.445313 36.546875 29.914063 11.230469 3.308593 21.496093 3.230469 32.550781.871093v40.449219c-87.300781-30.601562-151-114-151-211.800781 0-124.199219 101.800781-227 226-227s226 102.800781 226 227c0 97.800781-63.699219 181.199219-151 211.800781zm0 0"
                /></svg
            ></a
        >
    </p>
</footer>

<style>
    :global(body) {
        width: 100%;
        height: 100dvh;
        margin: 0;
        padding: 0;
        background: url(/background.svg);
        background-position: top;
        background-size: cover;
        background-repeat: no-repeat;
    }

    .no-underline {
        text-decoration: none;
    }

    footer {
        margin-top: 2rem;
        font-size: 0.75rem;
        text-align: center;
    }

    svg {
        width: 0.95em;
        height: 0.95em;
        margin-left: 0.5em;
    }

    p {
        margin: 0;
    }

    h1 {
        font-family: 'Righteous', serif;
        text-align: center;
        padding-top: 10rem;
        padding-bottom: 2rem;
    }

    main {
        max-width: 768px;
        margin: 0 auto;
        padding-inline: 2rem;
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

    button.copied {
        background-color: var(--pico-success);
    }

    .url-list {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        margin-top: 1rem;
    }

    .original-url {
        color: var(--pico-muted-color);
        font-size: 0.75em;
    }
    .error {
        color: #ff4444;
        background-color: rgba(255, 68, 68, 0.1);
        padding: 0.75rem 1rem;
        border-radius: 8px;
        margin: 1rem 0;
        border: 1px solid rgba(255, 68, 68, 0.3);
    }

    @media (max-width: 512px) {
        form {
            flex-direction: column;
        }
    }
</style>
