<script lang="ts">
    import { Link } from "svelte-routing";

    export let title: string;
    export let description: string;
    export let link: string;
</script>

<Link to={link}>
    <div
        class="p-6 h-full bg-neutral-600 shadow-lg rounded-2xl transition-transform transform hover:scale-105 hover:shadow-2xl cursor-pointer flex flex-col justify-between items-center"
    >
        <h3 class="font-bold text-2xl mb-2">{title}</h3>
        <p>{description}</p>
        <div class="mt-4">
            <button
                class="px-4 py-2 bg-neutral-900 text-white font-semibold rounded-lg transition-all cursor-pointer hover:bg-neutral-700"
            >
                Try Now
            </button>
        </div>
    </div>
</Link>
