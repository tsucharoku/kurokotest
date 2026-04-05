<template>
    <div class="news-page">
        <header class="news-page__header">
            <h1 class="news-page__title">お知らせ</h1>
        </header>

        <p v-if="pending" class="news-page__status">読み込み中…</p>
        <p v-else-if="error" class="news-page__status news-page__status--error">
            一覧を取得できませんでした。
        </p>
        <ul v-else-if="items.length" class="news-list">
            <li v-for="item in items" :key="item.topics_id" class="news-list__item">
                <NuxtLink :to="`/news/${item.topics_id}`" class="news-list__link">
                    <time :datetime="item.ymd" class="news-list__date">{{ formatDate(item.ymd) }}</time>
                    <span v-if="item.group_nm" class="news-list__category">{{ item.group_nm }}</span>
                    <span class="news-list__subject">{{ item.subject }}</span>
                </NuxtLink>
            </li>
        </ul>
        <p v-else class="news-page__status">お知らせはまだありません。</p>

        <nav
            v-if="!pending && !error && totalPages > 1"
            class="pagination"
            aria-label="ページ送り"
        >
            <NuxtLink
                v-if="currentPage > 1"
                :to="pageTo(currentPage - 1)"
                class="pagination__link"
            >
                前へ
            </NuxtLink>
            <span v-else class="pagination__edge pagination__edge--disabled">前へ</span>

            <span class="pagination__info">{{ currentPage }} / {{ totalPages }}</span>

            <NuxtLink
                v-if="currentPage < totalPages"
                :to="pageTo(currentPage + 1)"
                class="pagination__link"
            >
                次へ
            </NuxtLink>
            <span v-else class="pagination__edge pagination__edge--disabled">次へ</span>
        </nav>
    </div>
</template>

<script setup>
const PER_PAGE = 10;

const config = useRuntimeConfig();
const route = useRoute();

const currentPage = computed(() => {
    const raw = route.query.page;
    const s = Array.isArray(raw) ? raw[0] : raw;
    const n = typeof s === 'string' ? parseInt(s, 10) : NaN;
    return Number.isFinite(n) && n >= 1 ? n : 1;
});

const listUrl = computed(() => {
    const u = new URL('rcms-api/2/news', config.public.apiBase);
    u.searchParams.set('cnt', String(PER_PAGE));
    if (currentPage.value > 1) {
        u.searchParams.set('pageID', String(currentPage.value));
    }
    return u.toString();
});

const { data, pending, error } = await useFetch(listUrl, {
    credentials: 'include',
});

const items = computed(() => data.value?.list ?? []);

const totalPages = computed(() => data.value?.pageInfo?.totalPageCnt ?? 1);

function pageTo(page) {
    if (page <= 1) {
        return { path: '/' };
    }
    return { path: '/', query: { page: String(page) } };
}

function formatDate(ymd) {
    if (!ymd) return '';
    const [y, m, d] = ymd.split('-');
    if (!y || !m || !d) return ymd;
    return `${y}.${m}.${d}`;
}
</script>

<style scoped>
.news-page {
    max-width: 42rem;
    margin: 0 auto;
    padding: 2rem 1.25rem 3rem;
    font-family: system-ui, sans-serif;
    color: #1a1a1a;
}

.news-page__header {
    margin-bottom: 1.5rem;
    border-bottom: 1px solid #e5e5e5;
    padding-bottom: 0.75rem;
}

.news-page__title {
    margin: 0;
    font-size: 1.5rem;
    font-weight: 600;
    letter-spacing: 0.02em;
}

.news-page__status {
    margin: 0;
    color: #666;
    font-size: 0.9375rem;
}

.news-page__status--error {
    color: #b42318;
}

.news-list {
    list-style: none;
    margin: 0;
    padding: 0;
}

.news-list__item {
    border-bottom: 1px solid #eee;
}

.news-list__item:first-child {
    border-top: 1px solid #eee;
}

.news-list__link {
    display: grid;
    grid-template-columns: 6.5rem 5.5rem 1fr;
    gap: 0.75rem 1rem;
    align-items: baseline;
    padding: 1rem 0.25rem;
    text-decoration: none;
    color: inherit;
    transition: background-color 0.15s ease;
    border-radius: 6px;
    margin: 0 -0.25rem;
}

@media (max-width: 520px) {
    .news-list__link {
        grid-template-columns: 1fr;
        gap: 0.35rem;
    }
}

.news-list__link:hover {
    background-color: #f9f9f9;
}

.news-list__date {
    font-size: 0.8125rem;
    color: #666;
    font-variant-numeric: tabular-nums;
}

.news-list__category {
    font-size: 0.75rem;
    color: #444;
    background: #f0f0f0;
    padding: 0.2rem 0.5rem;
    border-radius: 4px;
    justify-self: start;
    line-height: 1.3;
}

.news-list__subject {
    font-size: 0.9375rem;
    font-weight: 500;
    line-height: 1.45;
}

.pagination {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-top: 2rem;
    padding-top: 1.25rem;
    border-top: 1px solid #e5e5e5;
}

.pagination__link {
    font-size: 0.875rem;
    color: #2563eb;
    text-decoration: none;
}

.pagination__link:hover {
    text-decoration: underline;
}

.pagination__info {
    font-size: 0.875rem;
    color: #444;
    font-variant-numeric: tabular-nums;
}

.pagination__edge {
    font-size: 0.875rem;
    color: #999;
}

.pagination__edge--disabled {
    cursor: default;
}
</style>
