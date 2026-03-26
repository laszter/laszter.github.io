<script lang="ts">
    import Card from "../lib/Card.svelte";
</script>

<main>
    <h1>Earth's Toolbox</h1>
    <h2>Featured Tools</h2>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 my-8">
        <Card
            link="/convert-property"
            title="⚒️ Property Converter"
            description="Convert private class members to public properties in C# code effortlessly."
        />
        <Card
            link="/json-format"
            title="📄 JSON Format & Minify"
            description="Format JSON and Minify"
        />
        <Card
            link="/pdf-unlock"
            title="🔓 Unlock PDF"
            description="Unlock password-protected PDF files"
        />
        <Card
            link="/error-stack-trace"
            title="🔥 Error Stack Trace Beautifier"
            description="Beautify and format error stack traces for better readability."
        />
        <Card
            link="/text-diff-compare"
            title="🧩 Text Diff Compare"
            description="Compare two texts side by side with highlighted additions, removals, and edits."
        />
    </div>
</main>
