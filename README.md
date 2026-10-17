# नेपाल न्युज (Nepal News)

React + TypeScript + Tailwind + Supabase. Public site, admin dashboard (/admin), Storage images, SEO functions for Vercel.
Nothing in this repo has been executed against a real Supabase project by the author. Run the checks below yourself.

## 1. Database (Supabase SQL Editor)
1. Supabase dashboard > your project > **SQL Editor** (left menu) > **New query**.
2. Paste ALL of `supabase/migrations/001_nepal_news.sql` > **Run**. Run once, on a fresh project.
3. Table Editor should show: profiles, categories (10 rows), articles. Storage should show bucket `article-images` (public).
4. Authentication > Providers > Email: **turn OFF "Allow new users to sign up"** (admins are created by hand). Even if left on, signups are plain readers.

## 2. Create the first admin (safe method)
1. Authentication > Users > **Add user** > Create new user. Enter email and a strong password; tick **Auto Confirm User**.
2. SQL Editor > run (with your email):
   `update public.profiles set role = 'admin' where id = (select id from auth.users where email = 'YOU@example.com');`
3. Log in at `/admin/login`. Role changes from the browser are impossible (column grant + trigger + RLS); only this SQL Editor step (or an existing admin) can promote.

## 3. Verify RLS
Run `supabase/tests/rls_tests.sql` in the SQL Editor. All rows must say PASS. It cleans up its own rows.
Tables/policies to eyeball (Authentication > Policies): RLS enabled on profiles, categories, articles; storage.objects has 4 `article-images:` policies.

## 4. Run the app
`cp .env.example .env`, fill VITE_SUPABASE_URL and VITE_SUPABASE_ANON_KEY (anon/publishable key only; NEVER the service_role key), then `npm install && npm run dev -- --host`.
Vercel: add the same variables plus SITE_URL. `api/` functions serve per-article meta tags, /sitemap.xml and /robots.txt.

## 5. Manual tests
- Logged out: open /admin -> redirected to /admin/login.
- Reader (sign up a second user in Dashboard, leave role reader): login shows "यो खाता admin होइन".
- Admin: create draft -> not visible on public site; publish -> visible; unpublish -> gone; duplicate slug -> error; delete -> confirm dialog.
- Images: upload JPG/PNG/WebP (stored as WebP <=1600px). Replace -> old file removed after save. Remove/delete article -> Storage > article-images > covers shows file gone. Try a .gif or renamed .txt -> rejected.
- SEO: view-source on a deployed /news/<slug> shows og:title, canonical, JSON-LD; /sitemap.xml and /robots.txt load.
