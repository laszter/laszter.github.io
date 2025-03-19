<script lang="ts">
  import { Router, Route } from "svelte-routing";
  import Home from "./Pages/Home.svelte";
  import ConvertProperty from "./Pages/ConvertProperty.svelte";
  import JsonFormat from "./Pages/JsonFormat.svelte";

  export let url = window.location.pathname;
</script>

<Router {url}>
  <Route path="/"><Home /></Route>
  <Route path="/convert-property"><ConvertProperty /></Route>
  <Route path="/json-format"><JsonFormat /></Route>
</Router>
<p class="text-neutral-500 inline-block mt-4">
  © 2024 Earth's Toolbox. All rights reserved.
</p>
