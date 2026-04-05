<template>
    <div class="category-page">
        <NuxtLink to="/" class="category-page__back">← お知らせ一覧</NuxtLink>

        <header class="category-page__header">
            <h1 class="category-page__title">カテゴリ一覧</h1>
        </header>

        <p v-if="pending" class="category-page__status">読み込み中…</p>
        <p v-else-if="error" class="category-page__status category-page__status--error">
            カテゴリを取得できませんでした。
        </p>
        <ul v-else-if="visibleCategories.length" class="category-list">
            <li v-for="cat in visibleCategories" :key="cat.topics_category_id" class="category-list__item">
                <NuxtLink
                    :to="{ path: '/', query: { category: String(cat.topics_category_id) } }"
                    class="category-list__link"
                >
                    <span class="category-list__name">{{ cat.category_nm }}</span>
                    <span v-if="cat.slug" class="category-list__slug">{{ cat.slug }}</span>
                </NuxtLink>
            </li>
        </ul>
        <p v-else class="category-page__status">カテゴリはまだありません。</p>
    </div>
</template>

<script setup>
const config = useRuntimeConfig();

const { data, pending, error } = await useFetch(
    `${config.public.apiBase}rcms-api/2/news/category`,
    {
        credentials: 'include',
    }
);

const visibleCategories = computed(() =>
    (data.value?.list ?? []).filter((c) => String(c.open_flg) !== '0')
);
</script>

<style scoped>
.category-page {
    max-width: 42rem;
    margin: 0 auto;
    padding: 2rem 1.25rem 3rem;
    font-family: system-ui, sans-serif;
    color: #1a1a1a;
}

.category-page__back {
    display: inline-block;
    margin-bottom: 1.25rem;
    font-size: 0.875rem;
    color: #2563eb;
    text-decoration: none;
}

.category-page__back:hover {
    text-decoration: underline;
}

.category-page__header {
    margin-bottom: 1.5rem;
    border-bottom: 1px solid #e5e5e5;
    padding-bottom: 0.75rem;
}

.category-page__title {
    margin: 0;
    font-size: 1.5rem;
    font-weight: 600;
    letter-spacing: 0.02em;
}

.category-page__status {
    margin: 0;
    color: #666;
    font-size: 0.9375rem;
}

.category-page__status--error {
    color: #b42318;
}

.category-list {
    list-style: none;
    margin: 0;
    padding: 0;
}

.category-list__item {
    border-bottom: 1px solid #eee;
}

.category-list__item:first-child {
    border-top: 1px solid #eee;
}

.category-list__link {
    display: flex;
    flex-wrap: wrap;
    align-items: baseline;
    gap: 0.5rem 1rem;
    padding: 1rem 0.25rem;
    text-decoration: none;
    color: inherit;
    border-radius: 6px;
    margin: 0 -0.25rem;
    transition: background-color 0.15s ease;
}

.category-list__link:hover {
    background-color: #f9f9f9;
}

.category-list__name {
    font-size: 1rem;
    font-weight: 500;
}

.category-list__slug {
    font-size: 0.8125rem;
    color: #888;
}
</style>
