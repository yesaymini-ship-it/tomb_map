# 🌿 掃墓地圖協作系統：Supabase 雲端資料庫設定指南Ω

本指南將帶您註冊並設定免費的 **Supabase** 雲端服務，讓您的地圖網頁能夠直接與雲端資料庫、雲端相簿相連，徹底擺脫本機電腦開機的限制。

---

## 步驟一：註冊並建立 Supabase 專案

1. 開啟 [Supabase 官網 (supabase.com)](ㄒ    )，點擊 **Sign Up** 註冊一個免費帳號（建議直接用 GitHub 帳號登入）。
2. 登入後，點擊 **New Project** 建立新專案：
   - **Organization**：選擇您的預設組織。
   - **Name**：輸入專案名稱（例如：`tomb-sweeping-map`）。
   - **Database Password**：設定您的資料庫密碼（請記下來，雖然這次專案網頁不會直接用到）。
   - **Region**：選擇靠近台灣的區域，建議選 **Tokyo (東京)** 或 **Singapore (新加坡)**，連線速度會最快。
   - **Pricing Plan**：選擇 **Free (免費)** 方案。
3. 點擊 **Create new project**，系統會開始配置您的雲端資料庫（大約需要 1-2 分鐘，請稍等它配置完成）。

---

## 步驟二：建立資料表與安全政策 (SQL Editor)

我們要利用 Supabase 的 SQL Editor 快速建立「地點資料表」與「允許大眾讀寫的權限（RLS Policy）」。

1. 在 Supabase 左側選單中，點選 **SQL Editor** 圖示（一個 `>_` 的符號）。
2. 點擊 **New query**（新建查詢）。
3. 複製下方所有的 SQL 程式碼，貼入編輯器中：

```sql
-- 1. 建立地點資料表
create table public.locations (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  ancestor text,
  description text,
  recorder text not null,
  lat double precision not null,
  lng double precision not null,
  photo_url text,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- 2. 開啟資料表的 RLS (資料列層級安全政策)
alter table public.locations enable row level security;

-- 3. 允許所有人讀取資料
create policy "Allow public read access"
  on public.locations for select
  using (true);

-- 4. 允許所有人新增資料
create policy "Allow public insert access"
  on public.locations for insert
  with check (true);
```

4. 貼上後，點擊右上角的 **Run** 按鈕。
5. 看到下方顯示 `Success` 代表資料庫表格已建立完畢！

---

## 步驟三：建立雲端相簿 (Storage)

我們需要一個存放墓地實拍照片的公開雲端硬碟空間。

1. 在 Supabase 左側選單中，點選 **Storage** 圖示（一個箱子的符號）。
2. 點擊 **New bucket**（新建儲存庫）：
   - **Bucket Name**：輸入 **`photos`**（請務必全小寫，與程式碼一致）。
   - **Public bucket**：**必須將此開關打開 (ON)**。這代表照片可以被公開讀取。
3. 點擊 **Save**。
4. 建立後，為了讓大家的手機能夠上傳照片，我們需要回到 **SQL Editor** 新增上傳權限：
   - 回到 **SQL Editor**，點選 **New query**。
   - 複製貼上並執行下方代碼：

```sql
-- 允許所有人上傳照片到 photos 儲存庫
create policy "Allow public upload"
  on storage.objects for insert
  with check (bucket_id = 'photos');

-- 允許所有人讀取 photos 儲存庫中的照片
create policy "Allow public read"
  on storage.objects for select
  using (bucket_id = 'photos');
```

   - 點擊 **Run** 執行。

---

## 步驟四：取得 API 金鑰並填入網頁

1. 在 Supabase 左下角點擊 **Project Settings**（齒輪圖示）。
2. 在左側子選單中選擇 **API**。
3. 尋找以下兩個資訊：
   - **Project URL** (例如：`https://xxxxxx.supabase.co`)
   - **`anon` `public` API Key** (這是一串很長的隨機英數字金鑰)
4. 開啟您的 [index_supabase.html](file:///Users/sayes/Downloads/geminicli/tomb_map/index_supabase.html)，滑到約 **第 471 行**，將這兩個值填入對應的變數中：

```javascript
// ==========================================
// ⚠️ 請填入您的 Supabase 連線資訊 (註冊後取得)
// ==========================================
const SUPABASE_URL = '您的 Project URL';
const SUPABASE_ANON_KEY = '您的 anon public API Key';
```

5. 儲存檔案。

---

## 步驟五：部署到 GitHub Pages (免費網頁空間)

當您填好金鑰並測試沒問題後，就可以上傳到 GitHub Pages 了：

1. 將原本的 [index.html](file:///Users/sayes/Downloads/geminicli/tomb_map/index.html) 備份或刪除。
2. 將 [index_supabase.html](file:///Users/sayes/Downloads/geminicli/tomb_map/index_supabase.html) 重新命名為 **`index.html`**。
3. 登入您的 GitHub 帳號，建立一個新的**公開 (Public)** Repository（例如命名為 `tomb_map`）。
4. 將專案中的 `index.html` 上傳到這個 Repository 中。
5. 在該 Repository 頁面中，點擊 **Settings** (設定) -> 左側選單的 **Pages**。
6. 在 **Build and deployment** 下方的 Branch 選擇 `main` (或 `master`)，資料夾選擇 `/ (root)`，然後點擊 **Save**。
7. 稍等 1-2 分鐘，GitHub 會為您產生一個專屬的網址（例如：`https://您的帳號.github.io/tomb_map/`）。

這時候，這個網址就是一個永久在線、免費且支援所有人定位和上傳照片的系統了！
