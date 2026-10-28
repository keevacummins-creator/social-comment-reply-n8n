# Social Comment AI Reply (n8n template)

This n8n workflow reads new comments on your Facebook Page, Instagram and YouTube channel. Claude works out what each person wants and writes a reply that sounds like your business. Leads and complaints get emailed to you.

## How it works

1. Facebook and Instagram comments arrive through a Meta webhook. YouTube is checked every 15 minutes, since YouTube has no webhook. TikTok and LinkedIn comments can come in too if you send them from a scraper (see setup step 7).
2. It skips comments it has already handled, and comments from the accounts you list in setup step 3, so it doesn't reply to itself.
3. Claude reads the comment and decides whether it's a lead, a question, praise, a complaint or spam. It writes a one to three sentence reply using the brand details you fill in. It's told not to make up prices or dates, and to skip spam and comments that only tag a friend.
4. Every comment is logged to an n8n data table.
5. On Facebook, Instagram and YouTube the reply is posted for you.
6. You get an email for leads, complaints, and TikTok/LinkedIn replies. TikTok and LinkedIn only allow comment replies through their APIs if you have a developer app they've approved (TikTok API for Business for Business accounts, LinkedIn's Community Management API for company pages). This template doesn't set those up, so for TikTok and LinkedIn the email has the reply ready to copy and paste.

## Accounts it works with

- **Facebook:** a Facebook Page you're an admin of. Personal profiles can't be connected through the API.
- **Instagram:** a Business or Creator account linked to that Facebook Page. Personal Instagram accounts can't be connected.
- **YouTube:** any channel. Connect it with the Google account that owns the channel.
- **TikTok and LinkedIn:** a scraper can read comments on public posts from any account. Replies are emailed to you to post yourself.

## What you'll need

- n8n with data tables and the Anthropic node. n8n Cloud has both. If you self-host, update n8n and make sure the webhook URL is reachable over HTTPS, because Meta won't send to anything else.
- An Anthropic API key. Each comment is one Claude call, billed to your Anthropic account.
- A Meta developer app (free) for Facebook and Instagram.
- A Google Cloud project with the YouTube Data API turned on, for YouTube.
- A paid scraper such as Apify, only if you want TikTok or LinkedIn.

## Setup

1. Import `social-comment-reply.json` into n8n.
2. Create a data table called `handled_comments` with these text columns: `commentId`, `platform`, `commentText`, `authorHandle`, `postId`, `timestamp`.
3. Open "Brand Voice (edit me)" and fill in your business name, how you sound, what you sell and the link to send people to, what it should never promise, and your own account IDs and handles.
4. Connect your own accounts on these nodes. n8n will ask for them when you open each one:
   - Anthropic API key, on "Claude Writes Reply"
   - SMTP, on "Email You" (or replace it with a Gmail or Outlook node), and change the from and to addresses
   - Facebook Graph API with a Page access token, on "Reply on Facebook" and "Reply on Instagram"
   - YouTube OAuth, on "Fetch YouTube Comments" and "Reply on YouTube"
5. Facebook and Instagram: in your Meta app, add a webhook with this workflow's production URL. Make up a verify token and paste the same one into "Verify Token Matches?" in place of `CHANGE_ME_VERIFY_TOKEN`. Subscribe to `feed` for your Page and `comments` for Instagram. Your Instagram account needs to be a Business or Creator account connected to your Facebook Page. The app needs `pages_read_engagement` and `pages_manage_engagement` for Facebook, and `instagram_basic` and `instagram_manage_comments` for Instagram. Until Meta approves those permissions through App Review, the app only sees comments from people who have a role on it, which is enough for testing.
6. YouTube: replace `YOUR_CHANNEL_ID` in "Fetch YouTube Comments". If you don't use YouTube, delete those nodes.
7. TikTok and LinkedIn (optional): run a comment scraper on a schedule (Apify has several) and POST each new comment to the same webhook URL as JSON:
   ```json
   {"platform": "tiktok", "commentId": "...", "text": "...", "author": "...", "authorId": "...", "postUrl": "..."}
   ```
8. Leave a comment from a second account to test it, then publish the workflow.

## Good to know

- Use a long-lived Page access token for the Facebook Graph API credential. The short-lived token Graph API Explorer gives you stops working within a few hours, and replies will start failing.
- If you made your own Google Cloud OAuth app for YouTube, set its publishing status to "In production". While it's set to "Testing", Google expires the login after 7 days and YouTube stops working.
- YouTube gives each project 10,000 API units a day. Each reply costs 50 and each check for new comments costs 1, so it can post just under 200 replies a day.
- The webhook URL accepts any POST request, so keep it private. Anyone who has it could send fake comments.
- Every new comment is one Claude call. On a busy account, check your Anthropic usage for the first week.
- If Claude's answer can't be read, nothing gets posted and you get a "Check this comment" email instead.
- YouTube is read 50 comments at a time, and only comments from the last hour are picked up, so turning it on won't reply to your old comments. If the workflow is off for more than an hour, comments from that gap are skipped.
- Every TikTok or LinkedIn comment that gets a reply sends you an email, because you have to post it yourself. On a busy account that's a lot of email.
- From Facebook, only new comments are used. Likes, edits, deletions and new posts are ignored.
- It uses `claude-sonnet-5`. You can switch to a cheaper model in "Claude Writes Reply" if the replies are still good enough.
