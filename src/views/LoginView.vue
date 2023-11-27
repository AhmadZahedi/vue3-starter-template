<template>
<header>
    <nav>
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
                    <span>masoud</span>
                    start
                </button>
            </template>

            <span>{{ $t('Hello') }}</span>
        </VTooltip>
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
    <div class="d-flex flex-column justify-content-between vh-100">
        <h1>{{ $t('Login') }}</h1>

        <div>
            <VTooltip
                trigger="click"
                position="bottom"
            >
                <template #activator="{ on }">
                    <button
                        v-on="on"
                        type="button"
                        class="btn btn-outline-warning text-black"
                    >
                        bottom
                    </button>
                </template>

                <span>{{ $t('Hello') }}</span>
            </VTooltip>
        </div>

        <div>
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
                        start
                    </button>
                </template>

                <span>{{ $t('Hello') }}</span>
            </VTooltip>
        </div>

        <div>
            <VTooltip
                trigger="click"
                position="end"
            >
                <template #activator="{ on }">
                    <button
                        v-on="on"
                        type="button"
                        class="btn btn-outline-warning text-black"
                    >
                        end
                    </button>
                </template>

                <span>{{ $t('HelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHello') }}</span>
            </VTooltip>
        </div>
    </div>

</main>
</template>
<script setup>
import VTooltip from "@/components/VTooltip.vue";
</script>
