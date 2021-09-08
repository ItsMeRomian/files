# 📄 files.dyna.host 📄

A Vue3 + Supabase + tailwind webapp where you can upload / delete / download files.  

## A Preview:

![image](https://user-images.githubusercontent.com/13468715/129405732-51134b47-df81-4b64-ae36-90b4f14efb53.png)

## How do I set this up?

### Setting up client

- Clone this repo
- Change the values in `lib/supabase.ts` to match your own settings, which can be found under `settings > API` in supabase.
- Run `npm i && npm run serve`

### Setting up Supabase Auth (Google)

- In Supabase, go to `settings > Auth` and enable Google Oauth. Follow [this](https://supabase.io/docs/learn/auth-deep-dive/auth-google-oauth) tutorial to generate tokens and secrets.
- Log in to **Files** by going to <http://localhost:3000>.
- Back in Supabase, go to the `Authentication` tab and copy your `UID`

### Setting up the storage bucket

- Go to `storage` and create a new bucket called `files`.
- Now click on `policies` and add a new policy.
- Click `make from scratch`. Check all the operations.
- Paste this in the box:

``` SQL
(
  (bucket_id = 'files':: text)
  AND (role() = 'authenticated':: text)
  AND (uid() = 'your_uid_here':: uuid)
)
```

- Change `your_uid_here` to your account's UID you found in the previous step.
- Upload a test file.
- Go back to **Files**, and you should see your test file.
