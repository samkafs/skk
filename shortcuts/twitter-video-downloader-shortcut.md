# iPhone Shortcut: Download Twitter/X Videos

This guide gives you a **ready-to-build Shortcut** that:

1. Accepts a shared Twitter/X post URL
2. Extracts the Tweet ID
3. Calls `api.fxtwitter.com` to get direct video links
4. Lets you pick a quality (if multiple videos exist)
5. Downloads and saves to Photos

---

## Important note

This shortcut uses a third-party endpoint (`api.fxtwitter.com`) to resolve media links.
If that service is down, rate-limited, or changes behavior, the shortcut may fail.

---

## Create the Shortcut

1. Open **Shortcuts** on iPhone
2. Tap **+** to create a new shortcut
3. Name it: **Download X Video**
4. Open shortcut settings (`i` icon):
   - Enable **Show in Share Sheet**
   - In "Accepted Types", keep **URLs** enabled

---

## Add actions (in this order)

### 1) Get URL from input
- Action: **Get URLs from Input**

### 2) Handle missing input
- Action: **If**
  - Condition: `URLs` `has any value`
  - If **true**: continue
  - Otherwise:
    - Action: **Ask for Input**
      - Prompt: `Paste a Twitter/X post link`
      - Input Type: `URL`
    - Action: **Set Variable**
      - Name: `InputURL`
      - Value: `Provided Input`
- In the `true` branch, add:
  - Action: **Get Item from List**
    - List: `URLs`
    - Item: `First Item`
  - Action: **Set Variable**
    - Name: `InputURL`
    - Value: `Item from List`

### 3) Extract Tweet ID with regex
- Action: **Match Text**
  - Text: `InputURL`
  - Pattern:
    ```regex
    (?i)(?:https?:\/\/)?(?:www\.)?(?:x|twitter)\.com\/[^\/]+\/status\/(\d+)
    ```

- Action: **If**
  - Condition: `Matches` `has any value`
  - If **false**:
    - Action: **Show Alert**
      - Title: `Invalid Link`
      - Message: `Could not find a Tweet status ID in this URL.`
    - Action: **Stop This Shortcut**

- In true branch:
  - Action: **Get Group from Matched Text**
    - Group Index: `1`
    - From: `First Match`
  - Action: **Set Variable**
    - Name: `TweetID`
    - Value: `Group`

### 4) Build API URL and fetch JSON
- Action: **Text**
  - Content:
    ```text
    https://api.fxtwitter.com/i/status/[TweetID]
    ```

- Action: **Set Variable**
  - Name: `ApiURL`
  - Value: `Text`

- Action: **Get Contents of URL**
  - URL: `ApiURL`
  - Method: `GET`
  - Headers:
    - `Accept` = `application/json`

- Action: **Set Variable**
  - Name: `ApiResponse`
  - Value: result of previous action

### 5) Validate API status
- Action: **Get Dictionary Value**
  - Dictionary: `ApiResponse`
  - Key: `code`
- Action: **Set Variable**
  - Name: `StatusCode`

- Action: **If**
  - Condition: `StatusCode` `is` `200`
  - If **false**:
    - Action: **Get Dictionary Value**
      - Dictionary: `ApiResponse`
      - Key: `message`
    - Action: **Show Alert**
      - Title: `Could not fetch media`
      - Message: `API returned [StatusCode]: [Dictionary Value]`
    - Action: **Stop This Shortcut**

### 6) Get video list
- Action: **Get Dictionary Value**
  - Dictionary: `ApiResponse`
  - Key: `tweet`
- Action: **Get Dictionary Value**
  - Dictionary: previous result
  - Key: `media`
- Action: **Get Dictionary Value**
  - Dictionary: previous result
  - Key: `videos`
- Action: **Set Variable**
  - Name: `Videos`

- Action: **If**
  - Condition: `Videos` `has any value`
  - If **false**:
    - Action: **Show Alert**
      - Title: `No video found`
      - Message: `This post may be text/photo only, private, or unsupported.`
    - Action: **Stop This Shortcut**

### 7) Let user choose one video URL
- Action: **List**
  - Leave empty
- Action: **Set Variable**
  - Name: `VideoURLs`
  - Value: result of `List`

- Action: **Repeat with Each**
  - Input: `Videos`
  - Inside repeat:
    - Action: **Get Dictionary Value**
      - Dictionary: `Repeat Item`
      - Key: `url`
    - Action: **Add to Variable**
      - Variable: `VideoURLs`
      - Value: result above

- Action: **Choose from List**
  - List: `VideoURLs`
  - Prompt: `Choose version to download`
- Action: **Set Variable**
  - Name: `SelectedVideoURL`
  - Value: chosen item

### 8) Download and save
- Action: **Get Contents of URL**
  - URL: `SelectedVideoURL`
  - Method: `GET`
- Action: **Save to Photo Album**
  - Media: downloaded file
- Action: **Show Notification**
  - Title: `Saved`
  - Body: `Video was saved to Photos.`

---

## Usage

1. In X/Twitter app, open a post with video
2. Tap Share -> Share via...
3. Choose **Download X Video**
4. Select preferred stream
5. Video saves to Photos

---

## Troubleshooting

- **Invalid Link**: make sure URL is a post/status URL (`.../status/<id>`)
- **API 401**: private or restricted post
- **API 404**: post not found/deleted/unsupported by endpoint
- **No video found**: tweet may contain only images, quote text, or external media

