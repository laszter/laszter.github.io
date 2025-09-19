<script lang="ts">
    import { Link } from "svelte-routing";

    let sourceText: string = "";

    let outputEl: HTMLDivElement;

    const beautifyStackTrace = () => {
        if (!outputEl) return;

        const lines = sourceText.replaceAll("\\r\\n", "\\n").split("\\n");

        outputEl.innerHTML = lines
            .map((line) =>
                line.toLowerCase().includes("controller")
                    ? `<span class="text-red-500 font-bold">${line}</span>`
                    : `<span>${line}</span>`,
            )
            .join("<br>");
    };
</script>

<main class="w-full inline-block">
    <div class="text-left">
        <Link to="/">
            <button
                class="cursor-pointer bg-neutral-900 hover:bg-neutral-700 py-2 px-4 text-white font-semibold rounded-lg transition-all"
                >🏠 Back</button
            >
        </Link>
    </div>
    <h1>🔥 Error Stack Trace Beautifier</h1>
    <div class="flex flex-col justify-between gap-4 w-full my-8">
        <textarea
            placeholder=""
            title="source"
            id="source-text"
            class="input-text"
            spellcheck="false"
            bind:value={sourceText}
            on:input={beautifyStackTrace}
        ></textarea>
        <div class="text-center text-neutral-500">↓</div>
        <div
            class="text-left bg-amber-200 text-black p-8 rounded-lg min-h-[200px] whitespace-pre-wrap break-words"
            bind:this={outputEl}
        ></div>
    </div>
</main>

<style>
    .input-text {
        width: 100%;
        height: 600px;
        box-sizing: border-box;
        resize: none;
        font-family: "Roboto", sans-serif;
        font-size: 16px;
        line-height: 1.5;
        background-color: #1f1f1f;
        color: #fff;
    }
</style>
