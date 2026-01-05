# cloud-ytdl

A reliable YouTube downloader and scraper using InnerTube clients. Includes video, stream, and Community Post extraction (text, images, polls). Production-ready for Node.js 18+.

**Created by:** [AlfiDev](https://github.com/cloudkuimages)  
**Repository:** [github.com/cloudkuimages/cloud-ytdl](https://github.com/cloudkuimages/cloud-ytdl)

## Features

- ✅ **Android InnerTube Client** - Reliable format extraction using official YouTube Android API
- ✅ **Automatic Signature Decoding** - Handles encrypted format URLs transparently
- ✅ **Restricted Videos** - Download age-restricted and region-locked videos with cookies
- ✅ **Community Posts** - Extract YouTube Community Posts (text, images, polls)
- ✅ **Subtitle Extraction** - Download subtitles/captions in XML or SRT format
- ✅ **Playlist Support** - Extract playlist information and video lists
- ✅ **Multi-format** - Supports various formats (MP4, WebM, M4A, etc.)
- ✅ **Stream Support** - Supports HLS and DASH streams
- ✅ **Production Ready** - Stable and tested with YouTube 2025

## Installation

```bash
npm install cloud-ytdl
```

## Quick Start

### Basic Video Download

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

// Download video with best quality
ytdl('https://www.youtube.com/watch?v=dQw4w9WgXcQ')
  .pipe(fs.createWriteStream('video.mp4'));
```

### Download with Quality Selection

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

// Highest quality
ytdl('VIDEO_URL', { quality: 'highest' })
  .pipe(fs.createWriteStream('video.mp4'));

// Specific format (1080p)
ytdl('VIDEO_URL', { quality: 137 })
  .pipe(fs.createWriteStream('video-1080p.mp4'));

// Audio only
ytdl('VIDEO_URL', { filter: 'audioonly' })
  .pipe(fs.createWriteStream('audio.m4a'));
```



## Complete API Documentation

### ytdl(url, [options])

Main function to download video from YouTube.

**Parameters:**
- `url` (string): Video URL or Video ID
- `options` (object): Download options (optional)

**Returns:** ReadableStream

**Example:**
```javascript
const stream = ytdl('https://www.youtube.com/watch?v=dQw4w9WgXcQ', {
  quality: 'highest',
  filter: 'audioandvideo'
});

stream.pipe(fs.createWriteStream('video.mp4'));
```

---

### ytdl.getInfo(url, [options])

Get complete video information without downloading.

**Parameters:**
- `url` (string): Video URL or Video ID
- `options` (object): Options (optional)

**Returns:** Promise<Object>

**Options:**
```javascript
{
  lang?: string,                    // Language (default: 'en')
  requestOptions?: {
    headers?: Record<string, string>, // Custom headers (e.g., cookies)
    timeout?: number,                 // Timeout in ms
    agent?: any,                      // Custom agent
    dispatcher?: any                  // Custom dispatcher
  }
}
```

**Example:**
```javascript
ytdl.getInfo('https://www.youtube.com/watch?v=dQw4w9WgXcQ')
  .then(info => {
    console.log('Title:', info.videoDetails.title);
    console.log('Channel:', info.videoDetails.author.name);
    console.log('Duration:', info.videoDetails.lengthSeconds, 'seconds');
    console.log('Format count:', info.formats.length);
    console.log('View count:', info.videoDetails.viewCount);
    
    // List all formats
    info.formats.forEach(format => {
      console.log(`ITAG ${format.itag}: ${format.qualityLabel} (${format.container})`);
    });
  })
  .catch(err => {
    console.error('Error:', err.message);
  });
```

**Response Object:**
```javascript
{
  videoDetails: {
    videoId: string,
    title: string,
    lengthSeconds: string,
    keywords: string[],
    channelId: string,
    viewCount: string,
    author: string,
    thumbnails: Array<{url: string, width: number, height: number}>,
    // ... other properties
  },
  formats: [
    {
      itag: number,
      url: string,
      mimeType: string,
      bitrate: number,
      width: number,
      height: number,
      qualityLabel: string,  // e.g., "720p", "1080p", "1440p", "2160p"
      container: string,
      hasVideo: boolean,
      hasAudio: boolean,
      codecs: string,
      // ... other properties
    }
  ],
  // ... other properties
}
```

---

### ytdl.getBasicInfo(url, [options])

Get basic video information (faster than getInfo).

**Parameters:**
- `url` (string): Video URL or Video ID
- `options` (object): Options (same as getInfo)

**Returns:** Promise<Object>

**Example:**
```javascript
ytdl.getBasicInfo('VIDEO_URL')
  .then(info => {
    console.log('Title:', info.videoDetails.title);
    console.log('Duration:', info.videoDetails.lengthSeconds);
  });
```

---

### ytdl.downloadFromInfo(info, [options])

Download video from a previously obtained info object.

**Parameters:**
- `info` (object): Info object from getInfo()
- `options` (object): Download options

**Returns:** ReadableStream

**Example:**
```javascript
// Get info first
const info = await ytdl.getInfo('VIDEO_URL');

// Choose format
const format = ytdl.chooseFormat(info.formats, { quality: 'highest' });

// Download with selected format
const stream = ytdl.downloadFromInfo(info, { format });

stream.pipe(fs.createWriteStream('video.mp4'));
```

---

### ytdl.chooseFormat(formats, [options])

Select format from format array based on quality and filter.

**Parameters:**
- `formats` (Array): Format array from info.formats
- `options` (object): Selection options

**Returns:** Object (format object)

**Options:**
```javascript
{
  quality?: 'lowest' | 'highest' | 'highestaudio' | 'lowestaudio' | 
           'highestvideo' | 'lowestvideo' | number | number[],
  filter?: 'audioandvideo' | 'videoandaudio' | 'video' | 'videoonly' | 
          'audio' | 'audioonly' | Function,
  format?: Object  // Specific format object
}
```

**Example:**
```javascript
const info = await ytdl.getInfo('VIDEO_URL');

// Select highest quality
const format1 = ytdl.chooseFormat(info.formats, { quality: 'highest' });

// Select best audio
const format2 = ytdl.chooseFormat(info.formats, { quality: 'highestaudio' });

// Select specific format (ITAG 137 = 1080p)
const format3 = ytdl.chooseFormat(info.formats, { quality: 137 });

// Select with custom filter
const format4 = ytdl.chooseFormat(info.formats, {
  quality: 'highest',
  filter: format => format.container === 'mp4' && format.hasAudio
});
```

---

### ytdl.filterFormats(formats, [filter])

Filter format array based on criteria.

**Parameters:**
- `formats` (Array): Format array
- `filter` (string | Function): Filter or custom function

**Returns:** Array (filtered formats)

**Available Filters:**
- `'audioandvideo'` or `'videoandaudio'` - Formats with video and audio
- `'video'` - Formats with video
- `'videoonly'` - Video formats without audio
- `'audio'` - Formats with audio
- `'audioonly'` - Audio-only formats

**Example:**
```javascript
const info = await ytdl.getInfo('VIDEO_URL');

// Filter audio only
const audioFormats = ytdl.filterFormats(info.formats, 'audioonly');

// Filter video only
const videoFormats = ytdl.filterFormats(info.formats, 'videoonly');

// Filter with custom function
const mp4Formats = ytdl.filterFormats(info.formats, format => 
  format.container === 'mp4'
);

// Filter 1080p or higher
const hdFormats = ytdl.filterFormats(info.formats, format => {
  const quality = parseInt(format.qualityLabel) || 0;
  return quality >= 1080;
});
```

---

### ytdl.getSubtitles(videoId, [options])

Get subtitles/captions for a video in XML or SRT format.

**Parameters:**
- `videoId` (string): Video ID or URL
- `options` (object): Options (optional)
  - `lang?: string` - Language code (default: 'id')
  - `format?: 'xml' | 'srt'` - Output format (default: 'xml')
  - `cookie?: string` - Cookie string for restricted videos

**Returns:** Promise<string | null>

**Example:**
```javascript
const ytdl = require('cloud-ytdl');

// Get subtitles in XML format (default)
ytdl.getSubtitles('dQw4w9WgXcQ', { lang: 'en' })
  .then(subtitle => {
    if (subtitle) {
      console.log('Subtitles:', subtitle);
      fs.writeFileSync('subtitles.xml', subtitle);
    } else {
      console.log('No subtitles available');
    }
  });

// Get subtitles in SRT format
ytdl.getSubtitles('dQw4w9WgXcQ', { 
  lang: 'en', 
  format: 'srt' 
})
  .then(subtitle => {
    if (subtitle) {
      fs.writeFileSync('subtitles.srt', subtitle);
    }
  });

// With cookies for restricted videos
const cookieString = 'VISITOR_INFO1_LIVE=xxx; YSC=yyy; ...';
ytdl.getSubtitles('VIDEO_ID', { 
  lang: 'en',
  format: 'srt',
  cookie: cookieString
})
  .then(subtitle => {
    if (subtitle) {
      fs.writeFileSync('subtitles.srt', subtitle);
    }
  });
```

**Note:** If no subtitles are found for the specified language, the function returns `null`.

---

### ytdl.getPlaylistInfo(url, [options])

Get playlist information including all videos in the playlist.

**Parameters:**
- `url` (string): Playlist URL
- `options` (object): Options (optional)
  - `agent?: Object` - Custom agent with cookies

**Returns:** Promise<Object>

**Example:**
```javascript
const ytdl = require('cloud-ytdl');

ytdl.getPlaylistInfo('https://www.youtube.com/playlist?list=PLxxxxx')
  .then(playlist => {
    console.log('Playlist ID:', playlist.id);
    console.log('Playlist Title:', playlist.title);
    console.log('Playlist Type:', playlist.type); // 'playlist' or 'radio'
    console.log('Video Count:', playlist.items.length);
    
    playlist.items.forEach((video, index) => {
      console.log(`${index + 1}. ${video.title}`);
      console.log(`   Video ID: ${video.videoId}`);
      console.log(`   Author: ${video.author}`);
      console.log(`   Duration: ${video.lengthSeconds} seconds`);
    });
  })
  .catch(err => {
    console.error('Error:', err.message);
  });
```

**Response Object:**
```javascript
{
  id: string,              // Playlist ID
  title: string,           // Playlist title
  type: 'playlist' | 'radio',  // Playlist type
  items: [                 // Array of videos
    {
      videoId: string,     // Video ID
      title: string,       // Video title
      author: string,      // Channel name
      lengthSeconds: number // Duration in seconds
    }
  ]
}
```

---

### ytdl.getPostInfo(url, [options])

Get YouTube Community Post information (text, images, polls).

**Parameters:**
- `url` (string): Community Post URL
- `options` (object): Options (optional)
  - `lang?: string` - Language
  - `headers?: Record<string, string>` - Custom headers

**Returns:** Promise<YTPostInfo>

**Example (CommonJS):**
```javascript
const { getPostInfo } = require('cloud-ytdl');

const postUrl = 'https://www.youtube.com/post/Ugkx...';
getPostInfo(postUrl)
  .then(post => {
    console.log('Author:', post.author);
    console.log('Author URL:', post.authorUrl);
    console.log('Published:', post.published);
    console.log('Content:', post.content);
    console.log('Images:', post.images);
    console.log('Likes:', post.likes);
    
    if (post.poll) {
      console.log('Poll Question:', post.poll.question);
      console.log('Poll Options:', post.poll.options);
      console.log('Total Votes:', post.poll.totalVotes);
    }
  })
  .catch(err => {
    console.error('Error:', err.message);
  });
```



**Response Object:**
```javascript
{
  author: string,              // Author/channel name
  authorUrl: string,           // Channel author URL
  published: string,           // Publish time (text)
  content: string,             // Post text content
  images: string[],            // Array of image URLs
  likes: string,               // Like count (text)
  poll?: {                     // Poll data (if available)
    question: string,          // Poll question
    options: Array<{           // Array of options
      text: string,            // Option text
      voteCount: number,       // Vote count
      isCorrect?: boolean      // Whether answer is correct (if available)
    }>,
    totalVotes: number,        // Total votes
    correctAnswer?: string | null  // Correct answer (if available)
  }
}
```

---

### ytdl.validateID(id)

Validate if string is a valid Video ID.

**Parameters:**
- `id` (string): Video ID to validate

**Returns:** boolean

**Example:**
```javascript
ytdl.validateID('dQw4w9WgXcQ');  // true
ytdl.validateID('invalid-id');    // false
```

---

### ytdl.validateURL(url)

Validate if string is a valid YouTube URL.

**Parameters:**
- `url` (string): URL to validate

**Returns:** boolean

**Example:**
```javascript
ytdl.validateURL('https://www.youtube.com/watch?v=dQw4w9WgXcQ');  // true
ytdl.validateURL('https://youtube.com/shorts/abc123');            // true
ytdl.validateURL('invalid-url');                                  // false
```

---

### ytdl.getURLVideoID(url)

Extract Video ID from YouTube URL.

**Parameters:**
- `url` (string): YouTube URL

**Returns:** string (Video ID)

**Throws:** Error if Video ID not found

**Example:**
```javascript
ytdl.getURLVideoID('https://www.youtube.com/watch?v=dQw4w9WgXcQ');  
// Returns: 'dQw4w9WgXcQ'

ytdl.getURLVideoID('https://youtu.be/dQw4w9WgXcQ');  
// Returns: 'dQw4w9WgXcQ'

ytdl.getURLVideoID('https://youtube.com/shorts/abc123def45');  
// Returns: 'abc123def45'
```

---

### ytdl.getVideoID(str)

Extract Video ID from string (can be URL or Video ID directly).

**Parameters:**
- `str` (string): URL or Video ID

**Returns:** string (Video ID)

**Throws:** Error if Video ID not found

**Example:**
```javascript
ytdl.getVideoID('dQw4w9WgXcQ');  // Returns: 'dQw4w9WgXcQ'
ytdl.getVideoID('https://www.youtube.com/watch?v=dQw4w9WgXcQ');  
// Returns: 'dQw4w9WgXcQ'
```

---

### ytdl.createAgent([cookies], [options])

Create custom agent with cookies for accessing restricted videos.

**Parameters:**
- `cookies` (Array | string): Array of cookie objects or cookie string
- `options` (object): Agent options

**Returns:** Object (agent object)

**Example:**
```javascript
// With cookie string
const cookieString = 'VISITOR_INFO1_LIVE=xxx; YSC=yyy; ...';
const agent = ytdl.createAgent(cookieString);

// With array cookies
const cookies = [
  { name: 'VISITOR_INFO1_LIVE', value: 'xxx', domain: '.youtube.com' },
  { name: 'YSC', value: 'yyy', domain: '.youtube.com' }
];
const agent2 = ytdl.createAgent(cookies);

// Use agent
ytdl('VIDEO_URL', { agent })
  .pipe(fs.createWriteStream('video.mp4'));
```

---

### ytdl.createProxyAgent(options, [cookies])

Create agent with proxy support.

**Parameters:**
- `options` (object | string): Proxy options (URI string or object)
- `cookies` (Array | string): Cookies (optional)

**Returns:** Object (agent object)

**Example:**
```javascript
// With proxy URI
const agent = ytdl.createProxyAgent('http://proxy.example.com:8080');

// With full options
const agent2 = ytdl.createProxyAgent({
  uri: 'http://proxy.example.com:8080',
  localAddress: '192.168.1.1'
}, cookieString);

// Use agent
ytdl('VIDEO_URL', { agent })
  .pipe(fs.createWriteStream('video.mp4'));
```

---

## Download Options (downloadOptions)

Options that can be used with `ytdl()` and `downloadFromInfo()`:

```javascript
{
  // Quality
  quality?: 'lowest' | 'highest' | 'highestaudio' | 'lowestaudio' | 
           'highestvideo' | 'lowestvideo' | number | number[],
  
  // Filter
  filter?: 'audioandvideo' | 'videoandaudio' | 'video' | 'videoonly' | 
          'audio' | 'audioonly' | Function,
  
  // Specific format
  format?: Object,
  
  // Request options
  requestOptions?: {
    headers?: Record<string, string>,  // Custom headers
    agent?: any,                        // Custom agent
    dispatcher?: any,                   // Custom dispatcher
    timeout?: number,                   // Timeout in ms
    maxRetries?: number,                // Maximum retries
    backoff?: {                         // Backoff strategy
      inc: number,
      max: number
    }
  },
  
  // Agent
  agent?: Object,                       // Custom agent with cookies
  
  // Range download
  range?: {
    start?: number,
    end?: number
  },
  
  // Live stream options
  begin?: string | number | Date,      // Start time for live stream
  liveBuffer?: number,                  // Buffer for live stream
  
  // Stream options
  highWaterMark?: number,               // Buffer size (default: 512KB)
  
  // IPv6 rotation
  IPv6Block?: string                    // IPv6 block for rotation
}
```

## Downloading Restricted Videos (Age-Restricted, Region-Locked)

For videos with restricted access (age-restricted, region-locked, or member-only), you need to provide YouTube cookies from a logged-in account.

### Method 1: Using Cookie Editor Extension (Recommended)

1. Install the [Cookie-Editor](https://cookie-editor.com/) extension:
   - [Chrome/Edge](https://chrome.google.com/webstore/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
   - [Firefox](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor/)

2. Open [YouTube](https://www.youtube.com) and log in to your account

3. Click the Cookie-Editor extension icon

4. Click "Export" → "Header String" (this will copy the cookie string to clipboard)

5. Use the cookie string in your code:

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

const cookieString = 'VISITOR_INFO1_LIVE=xxx; YSC=yyy; ...';

ytdl('VIDEO_URL', {
  requestOptions: {
    headers: {
      'Cookie': cookieString
    }
  }
}).pipe(fs.createWriteStream('video.mp4'));
```

### Method 2: Using Agent with Cookies

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

const cookieString = 'VISITOR_INFO1_LIVE=xxx; YSC=yyy; ...';
const agent = ytdl.createAgent(cookieString);

ytdl('VIDEO_URL', { agent })
  .pipe(fs.createWriteStream('video.mp4'));
```

### Method 3: Save Cookie to File

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

// Save cookie string to file (don't commit to git!)
fs.writeFileSync('.youtube-cookies.txt', 'YOUR_COOKIE_STRING');

// Read and use
const cookieString = fs.readFileSync('.youtube-cookies.txt', 'utf8');
const agent = ytdl.createAgent(cookieString);

ytdl('VIDEO_URL', { agent })
  .pipe(fs.createWriteStream('video.mp4'));
```

**⚠️ Security Warning:**
- Cookie strings contain authentication tokens. Treat them like passwords!
- Do not commit cookies to git repositories
- Add `.youtube-cookies.txt` to `.gitignore`
- Regenerate cookies if exposed (logout and login again)

**💡 Cookie Notes:**
- YouTube cookies typically last 1-2 weeks
- If downloads start failing with 403 errors, refresh your cookies

## Common ITAG Formats

### Video + Audio (Progressive)
- **18**: 360p MP4 (H.264, AAC)
- **22**: 720p MP4 (H.264, AAC) - Not always available

### Video Only (Adaptive)
- **133**: 240p MP4 (H.264)
- **134**: 360p MP4 (H.264)
- **135**: 480p MP4 (H.264)
- **136**: 720p MP4 (H.264)
- **137**: 1080p MP4 (H.264)
- **138**: 4320p MP4 (H.264) - Ultra high resolution
- **160**: 144p MP4 (H.264)
- **242**: 240p WebM (VP9)
- **243**: 360p WebM (VP9)
- **244**: 480p WebM (VP9)
- **247**: 720p WebM (VP9)
- **248**: 1080p WebM (VP9)
- **264**: 1440p MP4 (H.264)
- **266**: 2160p (4K) MP4 (H.264)
- **271**: 1440p WebM (VP9)
- **272**: 4320p WebM (VP9)
- **298**: 720p60 MP4 (H.264, 60fps)
- **299**: 1080p60 MP4 (H.264, 60fps)
- **302**: 720p60 HFR WebM (VP9, High Frame Rate)
- **303**: 1080p60 HFR WebM (VP9, High Frame Rate)
- **308**: 1440p60 HFR WebM (VP9, High Frame Rate)
- **313**: 2160p (4K) WebM (VP9)
- **315**: 2160p60 HFR WebM (VP9, High Frame Rate)
- **330-337**: HDR formats (144p-2160p with HDR and HFR)

### Audio Only (Adaptive)
- **139**: 48kbps M4A (AAC) - Lowest quality
- **140**: 128kbps M4A (AAC) - Medium quality
- **141**: 256kbps M4A (AAC) - High quality
- **249**: 48kbps WebM (Opus)
- **250**: 64kbps WebM (Opus)
- **251**: 160kbps WebM (Opus) - Highest quality

### Live Stream Formats
- **91-96**: HLS formats (144p-1080p)
- **127-128**: Audio-only HLS (AAC)

**Note:** Available formats depend on the video. Use `getInfo()` to see all available formats for a specific video.

## Events

Download stream emits standard Node.js events:

```javascript
const video = ytdl('VIDEO_URL');

video.on('info', (info, format) => {
  console.log('Downloading:', info.videoDetails.title);
  console.log('Format:', format.qualityLabel);
});

video.on('progress', (chunkLength, downloaded, total) => {
  const percent = ((downloaded / total) * 100).toFixed(2);
  console.log(`Progress: ${percent}%`);
});

video.on('data', (chunk) => {
  console.log('Received', chunk.length, 'bytes');
});

video.on('end', () => {
  console.log('Download complete!');
});

video.on('error', (error) => {
  console.error('Error:', error.message);
});

video.pipe(fs.createWriteStream('video.mp4'));
```

## Advanced Usage Examples

### Download with Progress Tracking

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

ytdl.getInfo('VIDEO_URL').then(info => {
  const format = ytdl.chooseFormat(info.formats, { quality: 'highest' });
  const video = ytdl.downloadFromInfo(info, { format });

  let downloaded = 0;
  const total = parseInt(format.contentLength);

  video.on('data', chunk => {
    downloaded += chunk.length;
    const percent = ((downloaded / total) * 100).toFixed(2);
    console.log(`Progress: ${percent}% (${(downloaded / 1024 / 1024).toFixed(2)} MB)`);
  });

  video.on('end', () => {
    console.log('Download complete!');
  });

  video.pipe(fs.createWriteStream('video.mp4'));
});
```

### Download Audio with Best Format

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

const url = 'VIDEO_URL';
const audioFormat = ytdl.filterFormats(
  await ytdl.getInfo(url).then(i => i.formats),
  'audioonly'
).sort((a, b) => (b.audioBitrate || 0) - (a.audioBitrate || 0))[0];

ytdl(url, { format: audioFormat })
  .pipe(fs.createWriteStream('audio.m4a'));
```

### Custom Filter Function

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

ytdl('VIDEO_URL', {
  filter: format => {
    return format.container === 'mp4' &&
           format.hasAudio &&
           format.hasVideo &&
           format.qualityLabel === '720p';
  }
}).pipe(fs.createWriteStream('video-720p.mp4'));
```

### Download Multiple Formats

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

const url = 'VIDEO_URL';
const info = await ytdl.getInfo(url);

// Download best video
const bestVideo = ytdl.chooseFormat(info.formats, { quality: 'highestvideo' });
ytdl.downloadFromInfo(info, { format: bestVideo })
  .pipe(fs.createWriteStream('video-only.mp4'));

// Download best audio
const bestAudio = ytdl.chooseFormat(info.formats, { quality: 'highestaudio' });
ytdl.downloadFromInfo(info, { format: bestAudio })
  .pipe(fs.createWriteStream('audio-only.m4a'));
```

### Get Subtitles

```javascript
const ytdl = require('cloud-ytdl');
const fs = require('fs');

async function downloadSubtitles() {
  try {
    // Get subtitles in SRT format
    const subtitle = await ytdl.getSubtitles('VIDEO_ID', {
      lang: 'en',
      format: 'srt'
    });
    
    if (subtitle) {
      fs.writeFileSync('subtitles.srt', subtitle);
      console.log('Subtitles downloaded successfully!');
    } else {
      console.log('No subtitles available for this video');
    }
  } catch (error) {
    console.error('Error:', error.message);
  }
}

downloadSubtitles();
```

### Get Playlist Information

```javascript
const ytdl = require('cloud-ytdl');

async function getPlaylist() {
  try {
    const playlist = await ytdl.getPlaylistInfo('PLAYLIST_URL');
    
    console.log('=== Playlist Info ===');
    console.log('Title:', playlist.title);
    console.log('ID:', playlist.id);
    console.log('Type:', playlist.type);
    console.log('Total Videos:', playlist.items.length);
    
    console.log('\n=== Videos ===');
    playlist.items.forEach((video, index) => {
      console.log(`${index + 1}. ${video.title}`);
      console.log(`   ID: ${video.videoId}`);
      console.log(`   Channel: ${video.author}`);
      console.log(`   Duration: ${video.lengthSeconds}s`);
    });
  } catch (error) {
    console.error('Error:', error.message);
  }
}

getPlaylist();
```

### Get Community Post Info

```javascript
const { getPostInfo } = require('cloud-ytdl');

async function getPost() {
  try {
    const post = await getPostInfo('https://www.youtube.com/post/Ugkx...');
    
    console.log('=== Community Post Info ===');
    console.log('Author:', post.author);
    console.log('Author URL:', post.authorUrl);
    console.log('Published:', post.published);
    console.log('Content:', post.content);
    console.log('Likes:', post.likes);
    console.log('Images:', post.images.length, 'images');
    
    if (post.poll) {
      console.log('\n=== Poll Info ===');
      console.log('Question:', post.poll.question);
      console.log('Total Votes:', post.poll.totalVotes);
      post.poll.options.forEach((option, index) => {
        console.log(`Option ${index + 1}: ${option.text} - ${option.voteCount} votes`);
      });
    }
  } catch (error) {
    console.error('Error:', error.message);
  }
}

getPost();
```

## Troubleshooting

### Error 403 Forbidden

**Problem:** Download fails with HTTP 403 error

**Solution:**
1. Add cookies from a logged-in YouTube account (see "Downloading Restricted Videos")
2. Check if video is region-locked or requires login
3. Refresh cookies if they have expired

### Format Not Found

**Problem:** Requested format/quality is not available

**Solution:** Check available formats first:
```javascript
ytdl.getInfo('VIDEO_URL').then(info => {
  console.log('Available formats:');
  info.formats.forEach(format => {
    console.log(`${format.itag}: ${format.qualityLabel} (${format.container})`);
  });
});
```

### Video Unavailable

**Problem:** Error "Video is unavailable"

**Possible Causes:**
- Video is private or deleted
- Video is region-locked (try with cookies from appropriate region)
- Live stream has ended
- Video is member-only (requires membership cookies)

### Signature Decoding Error

**Problem:** Error when decoding signature

**Solution:**
- Library automatically handles signature decoding
- If error occurs, try with valid YouTube cookies
- Make sure you're using the latest version of the library

## Differences from ytdl-core

This library is designed as a drop-in replacement for `ytdl-core` with the following improvements:

✅ **API Compatible** - Same API as ytdl-core, just change the require statement  
✅ **More Reliable** - Uses Android InnerTube client that works consistently  
✅ **Simpler** - No complex multi-client fallback logic  
✅ **Cleaner** - No browser automation or anti-bot detection needed  
✅ **Community Posts** - Support for YouTube Community Post extraction  
✅ **Subtitle Support** - Built-in subtitle/caption extraction  
✅ **Playlist Support** - Extract playlist information

### Migration from ytdl-core

```javascript
// Before
const ytdl = require('ytdl-core');

// After
const ytdl = require('cloud-ytdl');

// All existing code continues to work!
```

## Requirements

- Node.js 18 or higher
- npm or yarn

## License

MIT

## Credits

**Created by:** [AlfiDev](https://github.com/cloudkuimages)  
**Repository:** [github.com/cloudkuimages/cloud-ytdl](https://github.com/cloudkuimages/cloud-ytdl)

---

**Note:** This library uses Android InnerTube client for format extraction. Please use responsibly and in accordance with YouTube's Terms of Service.
