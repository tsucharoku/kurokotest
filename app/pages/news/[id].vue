<template>
    <div class="news-detail">
        <NuxtLink to="/" class="news-detail__back">← 一覧へ</NuxtLink>

        <p v-if="pending" class="news-detail__status">読み込み中…</p>
        <p v-else-if="error" class="news-detail__status news-detail__status--error">
            記事を取得できませんでした。
        </p>
        <article v-else-if="detail" class="news-detail__article">
            <header class="news-detail__header">
                <time :datetime="detail.ymd" class="news-detail__date">{{ formatDate(detail.ymd) }}</time>
                <span v-if="detail.group_nm" class="news-detail__category">{{ detail.group_nm }}</span>
                <h1 class="news-detail__title">{{ detail.subject }}</h1>
            </header>
            <div v-if="detail.contents" class="news-detail__body" v-html="detail.contents" />
            <p v-else class="news-detail__empty">本文はありません。</p>
        </article>
    </div>
</template>

<script setup>
const route = useRoute();
const config = useRuntimeConfig();

const id = computed(() => route.params.id);

const { data, pending, error } = await useFetch(
    () => `${config.public.apiBase}rcms-api/2/news/${id.value}`,
    {
        credentials: 'include',
        watch: [id],
    }
);

const detail = computed(() => data.value?.details ?? null);

function formatDate(ymd) {
    if (!ymd) return '';
    const [y, m, d] = ymd.split('-');
    if (!y || !m || !d) return ymd;
    return `${y}年${Number(m)}月${Number(d)}日`;
}
</script>

<style scoped>
.news-detail {
    max-width: 42rem;
    margin: 0 auto;
    padding: 2rem 1.25rem 3rem;
    font-family: system-ui, sans-serif;
    color: #1a1a1a;
}

.news-detail__back {
    display: inline-block;
    margin-bottom: 1.5rem;
    font-size: 0.875rem;
    color: #2563eb;
    text-decoration: none;
}

.news-detail__back:hover {
    text-decoration: underline;
}

.news-detail__status {
    margin: 0;
    color: #666;
    font-size: 0.9375rem;
}

.news-detail__status--error {
    color: #b42318;
}

.news-detail__header {
    margin-bottom: 1.5rem;
    padding-bottom: 1rem;
    border-bottom: 1px solid #e5e5e5;
}

.news-detail__date {
    display: block;
    font-size: 0.8125rem;
    color: #666;
    margin-bottom: 0.5rem;
    font-variant-numeric: tabular-nums;
}

.news-detail__category {
    display: inline-block;
    font-size: 0.75rem;
    color: #444;
    background: #f0f0f0;
    padding: 0.2rem 0.5rem;
    border-radius: 4px;
    margin-bottom: 0.75rem;
}

.news-detail__title {
    margin: 0;
    font-size: 1.375rem;
    font-weight: 600;
    line-height: 1.4;
    letter-spacing: 0.02em;
}

.news-detail__empty {
    margin: 0;
    color: #666;
    font-size: 0.9375rem;
}

.news-detail__body :deep(p) {
    margin: 0 0 1em;
    line-height: 1.7;
    font-size: 0.9375rem;
}

.news-detail__body :deep(p:last-child) {
    margin-bottom: 0;
}
</style>
