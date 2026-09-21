<!-- src/pages/LibraryPage.vue —— 资源库管理（对应 pages/library） -->
<template>
  <div class="page">
    <NavBar title="资源库管理" fallback="/" />

    <div class="pad">
      <!-- 搜索 -->
      <div class="search-wrap">
        <input
          v-model="keyword"
          class="search" type="search" placeholder="搜索标题 / 作者 / 词牌"
          @input="refresh" />
      </div>

      <!-- 列表 -->
      <div class="lib-list">
        <div v-for="p in list" :key="p.id" class="lib-item">
          <div class="lib-text">
            <div class="lib-title">{{ p.dispTitle }}</div>
            <div class="lib-meta muted">{{ p.author }} · {{ p.cipai }}</div>
          </div>
          <div class="lib-ops">
            <button class="op-btn" @click="openEdit(p.id)">编辑</button>
            <button class="op-btn del" @click="del(p.id)">删除</button>
          </div>
        </div>
        <div v-if="!list.length" class="muted empty">未找到匹配的诗词</div>
      </div>

      <button class="btn-primary add-btn" @click="openAdd">+ 添加新诗词</button>
    </div>

    <!-- 编辑表单 -->
    <div v-if="showForm" class="mask" @click.self="closeForm">
      <div class="form-box">
        <div class="form-title">{{ editing ? '编辑诗词' : '添加新诗词' }}</div>

        <label class="field">
          <span class="field-label">标题</span>
          <input v-model="form.title" class="field-input" placeholder="如：贺新郎·赋水仙" />
        </label>

        <label class="field">
          <span class="field-label">作者</span>
          <input v-model="form.author" class="field-input" placeholder="如：辛弃疾" />
        </label>

        <label class="field">
          <span class="field-label">词牌/体裁</span>
          <input v-model="form.cipai" class="field-input" placeholder="如：贺新郎" />
        </label>

        <label class="field">
          <span class="field-label">全文内容</span>
          <textarea v-model="form.content" class="field-area" rows="7" placeholder="请输入诗词全文"></textarea>
        </label>

        <div class="form-actions">
          <button class="btn-ghost f-act" @click="closeForm">取消</button>
          <button class="btn-primary f-act" @click="save">保存</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import NavBar from '../components/NavBar.vue';
import * as store from '../utils/store.js';
import * as poems from '../utils/poems.js';
import { toast, confirm } from '../utils/ui.js';

const list = ref([]);
const keyword = ref('');
const showForm = ref(false);
const editing = ref(false);
const form = ref({ id: '', title: '', author: '', cipai: '', content: '' });

function refresh() {
  const lib = store.getLibrary();
  const k = (keyword.value || '').trim();
  const filtered = k
    ? lib.filter((p) =>
        (p.title || '').indexOf(k) >= 0 ||
        (p.author || '').indexOf(k) >= 0 ||
        (p.cipai || '').indexOf(k) >= 0)
    : lib;
  const titleMap = poems.buildDisplayTitles(lib);
  list.value = filtered.map((p) => ({
    ...p,
    dispTitle: titleMap[p.id] || poems.displayTitle(p)
  }));
}

onMounted(refresh);

function openAdd() {
  showForm.value = true;
  editing.value = false;
  form.value = { id: '', title: '', author: '', cipai: '', content: '' };
}

function openEdit(id) {
  const p = store.getLibrary().find((x) => x.id === id);
  if (!p) return;
  showForm.value = true;
  editing.value = true;
  form.value = {
    id: p.id,
    title: p.title,
    author: p.author,
    cipai: p.cipai || '',
    content: p.content
  };
}

function closeForm() {
  showForm.value = false;
}

function save() {
  const f = form.value;
  if (!f.title.trim()) {
    toast('请填写标题');
    return;
  }
  if (!f.content.trim()) {
    toast('请填写全文');
    return;
  }
  const lib = store.getLibrary();
  const obj = {
    id: f.id || ('p' + Date.now()),
    title: f.title.trim(),
    author: f.author.trim(),
    cipai: f.cipai.trim(),
    content: f.content.trim()
  };
  if (f.id) {
    const i = lib.findIndex((x) => x.id === f.id);
    if (i >= 0) lib[i] = obj;
  } else {
    lib.push(obj);
  }
  store.setLibrary(lib);
  showForm.value = false;
  refresh();
}

async function del(id) {
  const ok = await confirm({
    title: '确认删除',
    content: '删除后不可恢复，确定要删除这首诗词吗？',
    confirmColor: '#b8860b'
  });
  if (!ok) return;
  const lib = store.getLibrary().filter((x) => x.id !== id);
  store.setLibrary(lib);
  refresh();
}
</script>

<style scoped>
.pad { padding-top: 4vw; }

.search-wrap { margin-bottom: 4vw; }

.search {
  width: 100%;
  background: var(--paper-card);
  border: 0.27vw solid var(--line);
  border-radius: 999px;
  padding: 3vw 4.4vw;
  font-size: 3.6vw;
  outline: none;
}

.search:focus { border-color: var(--gold); }

.lib-list { display: flex; flex-direction: column; gap: 2.4vw; }

.lib-item {
  display: flex;
  align-items: center;
  gap: 3vw;
  background: var(--paper-card);
  border: 0.27vw solid var(--line);
  border-radius: 1.6vw;
  padding: 3.2vw 3.4vw;
}

.lib-text { flex: 1; min-width: 0; }
.lib-title { font-size: 3.9vw; font-weight: 600; line-height: 1.45; word-break: break-all; }
.lib-meta { font-size: 3.1vw; margin-top: 0.8vw; }

.lib-ops { display: flex; gap: 2vw; flex: 0 0 auto; }

.op-btn {
  font-size: 3.2vw;
  color: var(--warm);
  background: #f6ecd6;
  border-radius: 999px;
  padding: 1.4vw 3vw;
}

.op-btn.del { color: var(--danger); background: #fbeaea; }

.empty { text-align: center; padding: 12vw 0; font-size: 3.6vw; }

.add-btn { margin-top: 6vw; font-size: 4.2vw; border-radius: 1.6vw; }

/* 表单 */
.form-box {
  width: 86vw;
  max-width: 420px;
  max-height: 84vh;
  overflow-y: auto;
  background: #fff;
  border-radius: 12px;
  padding: 20px 20px 18px;
}

.form-title {
  text-align: center;
  font-size: 16px;
  font-weight: 700;
  color: var(--ink-green);
  margin-bottom: 16px;
}

.field { display: block; margin-bottom: 14px; }

.field-label {
  display: block;
  font-size: 13px;
  color: var(--warm);
  margin-bottom: 6px;
}

.field-input, .field-area {
  width: 100%;
  border: 1px solid var(--line);
  border-radius: 8px;
  padding: 9px 12px;
  font-size: 14px;
  outline: none;
  background: #fffdf9;
  color: #2b2b2b;
}

.field-input:focus, .field-area:focus { border-color: var(--gold); }
.field-area { resize: vertical; line-height: 1.7; }

.form-actions { display: flex; gap: 12px; margin-top: 6px; }
.f-act { flex: 1; font-size: 15px; padding: 11px 0; }

@media (min-width: 640px) {
  .pad { max-width: 640px; margin: 0 auto; }
  .search { font-size: 14px; padding: 11px 18px; }
  .lib-title { font-size: 15px; }
  .lib-meta { font-size: 12.5px; }
  .op-btn { font-size: 13px; padding: 5px 12px; }
  .empty { font-size: 14px; }
  .add-btn { font-size: 16px; padding: 13px 0; }
}
</style>
