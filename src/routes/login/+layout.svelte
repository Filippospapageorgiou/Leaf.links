<script lang="ts">
    import { page } from '$app/stores';
    import AnimatedRoute from '$lib/components/AnimatedRoute.svelte';
</script>

<nav class="flex justify-center my-6">
    <ul class="steps">
        <a href="/login" class="step step-primary">Sign In</a>
        <a 
           href="/login/username"
           class="step"
           class:step-primary={$page.route.id?.match(/username|photo/g)}>
           Choose Username
        </a>
        <a 
            href="/login/photo"
            class="step"
            class:step-primary={$page.route.id?.includes("photo")}>
            Add Photo
        </a>    
    </ul>
</nav>

<!--
with slots any child pages can be rendered inside the layout
in the main and div tags taht we gave wraped the slot-->

<AnimatedRoute>
<main class="card w-4/6 bg-neutral text-neutral-content mx-auto">
    <div class="card-body items-center text-center">
        <slot/>
    </div>
</main>
</AnimatedRoute>



