<script lang="ts">
    import UserLink from "$lib/components/UserLink.svelte";
    import type { PageData } from "./$types";
    import { userData } from "$lib/firebase";

    export let data: PageData;
</script>

<svelte:head>
    <title>@{data.username} Links</title>
    <meta name="description" content={data.bio} />
</svelte:head>

<main class=" prose text-center mx-auto mt-8">

    <h1 class="text-7xl text-purple-500 mb-5">@{data.username}</h1>

    <img 
        src={data.photoURL ?? "/favicon.png"}
        alt="photoURL"
        width="256"
        class="mx-auto my-8"    
    />

    <p class="text-xl my-8">{data.bio ?? "no bio yet"}</p>
    <ul class="list-one">
        {#each data.links as item}
            <li class="my-2">
                <UserLink {...item} />
            </li>    
        {/each}
    </ul>
</main>