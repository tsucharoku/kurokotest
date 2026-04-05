<template>
    <div class="news-page">
        <header class="news-page__header">
            <h1 class="news-page__title">お知らせ</h1>
            <NuxtLink to="/news/categories" class="news-page__nav-link">カテゴリ一覧</NuxtLink>
        </header>

        <form class="news-search" @submit.prevent="submitSearch">
            <label class="news-search__label" for="news-search-input">タイトルで検索</label>
            <div class="news-search__row">
                <input
                    id="news-search-input"
                    v-model="searchInput"
                    class="news-search__input"
                    type="search"
                    name="q"
                    placeholder="キーワードを入力"
                    autocomplete="off"
                />
                <button type="submit" class="news-search__submit">検索</button>
            </div>
            <p v-if="searchRaw" class="news-search__hint">
                「{{ searchRaw }}」で絞り込み
                <NuxtLink :to="listLinkWithoutSearch" class="news-search__clear">検索を解除</NuxtLink>
            </p>
            <p v-if="categoryId" class="news-search__hint">
                カテゴリで絞り込み中（ID: {{ categoryId }}）
                <NuxtLink :to="listLinkWithoutCategory" class="news-search__clear">カテゴリを解除</NuxtLink>
            </p>
        </form>

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
const router = useRouter();

const currentPage = computed(() => {
    const raw = route.query.page;
    const s = Array.isArray(raw) ? raw[0] : raw;
    const n = typeof s === 'string' ? parseInt(s, 10) : NaN;
    return Number.isFinite(n) && n >= 1 ? n : 1;
});

const searchRaw = computed(() => {
    const raw = route.query.q;
    const s = Array.isArray(raw) ? raw[0] : raw;
    return typeof s === 'string' ? s.trim() : '';
});

const categoryId = computed(() => {
    const raw = route.query.category;
    const s = Array.isArray(raw) ? raw[0] : raw;
    if (typeof s !== 'string' || !/^\d+$/.test(s)) {
        return null;
    }
    return s;
});

const listLinkWithoutCategory = computed(() => {
    const query = { ...route.query };
    delete query.category;
    delete query.page;
    if (Object.keys(query).length === 0) {
        return { path: '/' };
    }
    return { path: '/', query };
});

const listLinkWithoutSearch = computed(() => {
    const query = { ...route.query };
    delete query.q;
    delete query.page;
    if (Object.keys(query).length === 0) {
        return { path: '/' };
    }
    return { path: '/', query };
});

const searchInput = ref(searchRaw.value);

watch(
    () => route.query.q,
    (q) => {
        const s = Array.isArray(q) ? q[0] : q;
        searchInput.value = typeof s === 'string' ? s : '';
    }
);

function submitSearch() {
    const q = searchInput.value.trim();
    const query = { ...route.query };
    if (q) {
        query.q = q;
    } else {
        delete query.q;
    }
    delete query.page;
    router.push({ path: '/', query });
}

function escapeKurocoFilterString(value) {
    return value.replace(/\\/g, '\\\\').replace(/"/g, '\\"');
}

/** @see https://kuroco.app/ja/docs/reference/filter-query/ */
function buildTitleFilterParam(query) {
    const inner = escapeKurocoFilterString(query);
    return `(subject icontains "${inner}")`;
}

function buildContentsTypeFilter(id) {
    return `(contents_type = ${id})`;
}

function buildListFilterParam() {
    const q = searchRaw.value;
    const cat = categoryId.value;
    if (q && cat) {
        return `(${buildTitleFilterParam(q)} AND ${buildContentsTypeFilter(cat)})`;
    }
    if (q) {
        return buildTitleFilterParam(q);
    }
    if (cat) {
        return buildContentsTypeFilter(cat);
    }
    return null;
}

function buildListUrl(page, filterParam) {
    const u = new URL('rcms-api/2/news', config.public.apiBase);
    u.searchParams.set('cnt', String(PER_PAGE));
    if (page > 1) {
        u.searchParams.set('pageID', String(page));
    }
    if (filterParam) {
        u.searchParams.set('filter', filterParam);
    }
    return u.toString();
}

const { data, pending, error } = await useAsyncData(
    'news-list',
    async () => {
        const page = currentPage.value;
        const filterParam = buildListFilterParam();
        return await $fetch(buildListUrl(page, filterParam), { credentials: 'include' });
    },
    {
        watch: [currentPage, searchRaw, categoryId],
    }
);

const items = computed(() => data.value?.list ?? []);

const totalPages = computed(() => data.value?.pageInfo?.totalPageCnt ?? 1);

function pageTo(page) {
    const query = {};
    if (searchRaw.value) {
        query.q = searchRaw.value;
    }
    if (categoryId.value) {
        query.category = categoryId.value;
    }
    if (page > 1) {
        query.page = String(page);
    }
    if (Object.keys(query).length === 0) {
        return { path: '/' };
    }
    return { path: '/', query };
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
    display: flex;
    flex-wrap: wrap;
    align-items: baseline;
    justify-content: space-between;
    gap: 0.75rem 1rem;
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

.news-page__nav-link {
    font-size: 0.875rem;
    font-weight: 500;
    color: #2563eb;
    text-decoration: none;
}

.news-page__nav-link:hover {
    text-decoration: underline;
}

.news-search {
    margin-bottom: 1.5rem;
}

.news-search__label {
    display: block;
    font-size: 0.8125rem;
    font-weight: 500;
    color: #444;
    margin-bottom: 0.35rem;
}

.news-search__row {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    align-items: stretch;
}

.news-search__input {
    flex: 1 1 12rem;
    min-width: 0;
    padding: 0.5rem 0.65rem;
    font-size: 0.9375rem;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-family: inherit;
}

.news-search__input:focus {
    outline: 2px solid #93c5fd;
    outline-offset: 1px;
    border-color: #2563eb;
}

.news-search__submit {
    padding: 0.5rem 1rem;
    font-size: 0.875rem;
    font-weight: 500;
    color: #fff;
    background: #2563eb;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-family: inherit;
}

.news-search__submit:hover {
    background: #1d4ed8;
}

.news-search__hint {
    margin: 0.5rem 0 0;
    font-size: 0.8125rem;
    color: #666;
}

.news-search__clear {
    margin-left: 0.5rem;
    font-size: 0.8125rem;
    color: #2563eb;
    text-decoration: none;
}

.news-search__clear:hover {
    text-decoration: underline;
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
