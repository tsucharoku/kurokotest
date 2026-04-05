<template>
    <div>
        <div>{{ response.details.subject }}</div>
    </div>
</template>

<script setup>
const config = useRuntimeConfig();

const { data: response } = await useFetch(
    `${config.public.apiBase}rcms-api/2/news/3`,
    {
        credentials: 'include',
    }
);
</script>