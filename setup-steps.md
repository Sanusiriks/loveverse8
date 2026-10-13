> **Gmail (Google) button abhi band hai** (`google:false`). App email + strong password se chalta hai. Baad me Gmail chahiye to `LOVEVERSE_CONFIG` me `google:true` karo aur section B (Google Cloud) poora karo.

# LoveVerse – Real login ke liye setup (bina terminal ke)

## A. Supabase (10 min)
1. supabase.com par free account → New project (region: Mumbai/Singapore, strong database password).
2. Left menu **SQL Editor** → New query → `SETUP_ALL.sql` ka poora text paste → Run.
3. **Project Settings → API**: "Project URL" aur "anon public" key copy karo. (service_role key kabhi kisi ko mat do.)
4. **Authentication → Providers → Email**: Confirm email ON, minimum password length 10.
5. **Authentication → Attack Protection**: CAPTCHA ON (Cloudflare Turnstile free hai).

## B. Gmail (Google) login (10 min)
1. console.cloud.google.com → naya project → **APIs & Services → OAuth consent screen** (External; app name LoveVerse; apna email; Privacy aur Terms URL baad me daalna).
2. **Credentials → Create credentials → OAuth client ID → Web application**.
   - Authorized redirect URI: `https://<PROJECT-REF>.supabase.co/auth/v1/callback` (Project URL ka pehla hissa).
   - Authorized JavaScript origin: aapki site ka URL.
3. Client ID aur Secret copy → Supabase **Authentication → Providers → Google** me paste → Enable.
4. Supabase **Authentication → URL Configuration**: Site URL = aapki site; Redirect URLs me wahi URL.
5. Google consent screen ko "Publish app" karo, warna sirf test users login kar paayenge.

## C. Site deploy (5 min)
1. `index.html` me ye line dhundo aur apni values bharo:
   `window.LOVEVERSE_CONFIG={url:"https://xxxx.supabase.co",anon:"<anon key>",priceM:"₹149",priceY:"₹999"}`
2. app.netlify.com → Add new site → Deploy manually → `loveverse-deploy.zip` ka folder drag-drop. (HTTPS apne-aap milta hai.)
3. Netlify ka URL Supabase aur Google me (B.2 aur B.4) daal do.

## D. Pehla admin
Site par sign up karo → email confirm → Supabase **SQL Editor**:
`update users set role='admin' where id = (select id from auth.users where email='aapka@gmail.com');`

## E. AI aur payments (baad me, optional)
Supabase dashboard → **Edge Functions** → Create function: `generate`, `delete-account`, `billing`, `stripe-webhook`, `admin-users` (code `supabase/functions/*/index.ts` se copy). Secrets: Edge Functions → Secrets (ANTHROPIC_API_KEY, STRIPE_*). `stripe-webhook` me "Verify JWT" OFF karo.

## F. Test (zaroor)
Do alag accounts banao: A ka data B ko nahi dikhna chahiye; normal user admin panel nahi khol paaye; galat password 5 baar → 60 second lock.

## G. Naye plans aur account limit (zaroor)
SQL Editor me `0008_signup_guard.sql` aur `0009_tiers.sql` run karo (ya naya `SETUP_ALL.sql`).
Phir Supabase -> Authentication -> Hooks -> **Before User Created** -> `hook_before_user_created` chuno.
CAPTCHA: Cloudflare Turnstile ka site key `LOVEVERSE_CONFIG.captcha` me, secret Supabase CAPTCHA settings me.
Details: `loveverse-backend-setup.md` section 10 aur 11.
