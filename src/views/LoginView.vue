<template>
<header>
    <nav>
        <RouterLink :to="{ name: 'Dashboard' }">
            {{ $t('Dashboard') }}
        </RouterLink>
        |
        <RouterLink :to="{ name: 'Login' }">
            {{ $t('Login') }}
        </RouterLink>
    </nav>
</header>

<main>
    <h1>{{ $t('Login') }}</h1>

    <div style="height: 4000px; margin: 500px">
        <VTooltip
            trigger="click"
            position="start"
        >
            <template #activator="{ on }">
                <button
                    v-on="on"
                    type="button"
                    class="btn btn-outline-warning text-black"
                >
                    click here
                </button>
            </template>

            <span>{{ $t('Hello') }}</span>
        </VTooltip>
    </div>
</main>
</template>
<script setup>
import VTooltip from "@/components/VTooltip.vue";
</script>
