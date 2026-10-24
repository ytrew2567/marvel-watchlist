# 漫威影視追劇清單

以繁體中文整理漫威相關電影與影集的追劇清單，可記錄觀看進度、篩選作品，並透過 Supabase 帳號在不同裝置同步。

## 功能

- 依上映／首播順序瀏覽電影、影集、短片與動畫作品。
- 查看官方故事時間線，或切換成只看電影。
- 搜尋作品名稱，依作品類型與觀看狀態篩選。
- 使用「只看必看」及「只看未看」快速縮小清單。
- 勾選觀看進度並查看整體進度；篩選不會改變總進度計算。
- 載入 TMDb 海報，並顯示 TMDb 標誌與資料來源聲明。
- 未登入時將進度保存在目前瀏覽器；登入後可同步至 Supabase。

## GitHub Pages 發布

1. 將完整網站檔命名為 `index.html`，放在 GitHub 儲存庫根目錄。
2. 在儲存庫的 **Settings → Pages** 設定發布來源，選擇主要分支及 `/(root)`。
3. 儲存後開啟 GitHub Pages 網址。更新 `index.html` 並提交後，重新整理網站查看最新版本。

## TMDb 海報設定

網站透過 TMDb API 搜尋作品並顯示海報。將 TMDb 的 **API Key (v3 auth)** 填入 `index.html` 中的 `TMDB_API_KEY` 設定。API Read Access Token 是另一種憑證；若使用 Token，程式需以 Bearer 標頭傳送。

TMDb API Key 會隨前端程式提供給瀏覽器，因此不應視為私密密碼。若使用量異常，請至 TMDb 帳戶設定檢查憑證。請保留頁面上的 TMDb 標誌及以下聲明：

> 本產品使用 TMDB API，但未獲 TMDB 背書或認證。  
> This product uses the TMDB API but is not endorsed or certified by TMDB.

## Supabase 跨裝置同步

網站使用 Supabase Auth 電子郵件登入，並將每個帳號的勾選進度存於 `watch_progress` 資料表。請在 `index.html` 設定 Supabase Project URL 和 Publishable key。

資料表需包含 `user_id`、`done`、`updated_at` 欄位，並啟用 Row Level Security (RLS)。以下 SQL 可建立與目前網站相容的資料表及政策：

```sql
create table if not exists public.watch_progress (
  user_id uuid primary key references auth.users(id) on delete cascade,
  done jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.watch_progress enable row level security;

create policy "Users can read their own watch progress"
on public.watch_progress for select
using (auth.uid() = user_id);

create policy "Users can insert their own watch progress"
on public.watch_progress for insert
with check (auth.uid() = user_id);

create policy "Users can update their own watch progress"
on public.watch_progress for update
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

在 Supabase Auth 的 URL 設定中，將 GitHub Pages 網址加入允許的 **Site URL** 與 **Redirect URLs**。電子郵件登入連結能否送達，取決於 Supabase 電子郵件寄送設定；正式使用建議設定自己的 SMTP 服務。

### 設定注意事項

- Supabase Publishable key（舊稱 anon key）可放在前端，但資料安全必須依靠 RLS 政策；不要在前端放入 `service_role` key。
- 使用同一個登入帳號可在不同裝置同步；未登入時的進度只存在該裝置瀏覽器。
- 初次登入時，網站會嘗試合併此瀏覽器的既有進度與雲端進度。跨瀏覽器或跨裝置使用前，建議先登入並確認同步狀態。

## 資料與同步限制

- 網站是 GitHub Pages 上的靜態前端，沒有伺服器端程式執行環境；`claude.use` 無法在 GitHub Pages 上執行。跨裝置同步由 Supabase 提供。
- TMDb 搜尋結果依作品名稱比對，少數別名、翻譯片名或特殊篇章可能找不到海報或配對不完全。
- 進度以 Supabase 帳號為單位；清除瀏覽器資料可能移除尚未同步的本機進度。

## 技術

- HTML、CSS、原生 JavaScript
- GitHub Pages
- Supabase Auth 與資料庫
- TMDb API

## 版權與致謝

作品名稱、角色與相關商標屬各自權利人所有。本清單為非官方個人整理。海報與影視資料由 TMDb API 提供；本產品未獲 TMDb 背書或認證。
