# Social Comment AI Reply (n8n template)

This n8n workflow reads new comments on your Facebook Page, Instagram and YouTube channel. Claude works out what each person wants and writes a reply that sounds like your business. Leads and complaints get emailed to you.

## How it works

1. Facebook and Instagram comments arrive through a Meta webhook. YouTube is checked every 15 minutes, since YouTube has no webhook. TikTok and LinkedIn comments can come in too if you send them from a scraper (see setup step 6).
2. It skips comments it has already handled, and comments from your own accounts, so it never replies to itself.
3. Claude reads the comment and decides whether it's a lead, a question, praise, a complaint or spam. It writes a one to three sentence reply using the brand details you fill in. It won't make up prices or dates, and it doesn't reply to spam or comments that only tag a friend.
4. Every comment is logged to an n8n data table.
5. On Facebook, Instagram and YouTube the reply is posted for you.
6. You get an email for leads, complaints, and TikTok/LinkedIn replies. TikTok and LinkedIn don't let normal accounts post comment replies through their APIs, so for those the email has the reply ready to copy and paste.

## Setup

1. Import `social-comment-reply.json` into n8n.
2. Create a data table called `handled_comments` with these text columns: `commentId`, `platform`, `commentText`, `authorHandle`, `postId`, `timestamp`.
3. Open "Brand Voice (edit me)" and fill in your business name, how you sound, what you sell and the link to send people to, what it should never promise, and your own account IDs and handles.
4. Connect your own accounts on these nodes. n8n will ask for them when you open each one:
   - Anthropic API key, on "Claude Writes Reply"
   - SMTP, on "Email You" (or replace it with a Gmail or Outlook node), and change the from and to addresses
   - Facebook Graph API with a Page access token, on "Reply on Facebook" and "Reply on Instagram"
   - YouTube OAuth, on "Fetch YouTube Comments" and "Reply on YouTube"
5. Facebook and Instagram: in your Meta app, add a webhook with this workflow's production URL. Make up a verify token and paste the same one into "Verify Token Matches?" in place of `CHANGE_ME_VERIFY_TOKEN`. Subscribe to `feed` for your Page and `comments` for Instagram. The app needs the `pages_read_engagement`, `pages_manage_engagement` and `instagram_manage_comments` permissions, and has to be in Live mode to receive comments from people other than you.
6. YouTube: replace `YOUR_CHANNEL_ID` in "Fetch YouTube Comments". If you don't use YouTube, delete those nodes.
7. TikTok and LinkedIn (optional): run a comment scraper on a schedule (Apify has several) and POST each new comment to the same webhook URL as JSON:
   ```json
   {"platform": "tiktok", "commentId": "...", "text": "...", "author": "...", "authorId": "...", "postUrl": "..."}
   ```
8. Leave a comment from a second account to test it, then publish the workflow.

## Good to know

- The webhook URL accepts any POST request, so keep it private. Anyone who has it could send fake comments.
- Every new comment is one Claude call. On a busy account, check your Anthropic usage for the first week.
- If Claude's answer can't be read, nothing gets posted and you get a "Check this comment" email instead.
- YouTube is read 50 comments at a time, and only comments from the last hour are picked up, so turning it on won't reply to your old comments. If the workflow is off for more than an hour, comments from that gap are skipped.
- Every TikTok or LinkedIn comment that gets a reply sends you an email, because you have to post it yourself. On a busy account that's a lot of email.
- Facebook only lets through new comments. Likes, edits, deletions and your own posts are ignored.
- It uses `claude-sonnet-5`. You can switch to a cheaper model in "Claude Writes Reply" if the replies are still good enough.
