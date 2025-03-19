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
    </div>
</main>
