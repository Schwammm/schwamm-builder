<div  align="center">
<img
src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=30&pause=1500&color=3B82F6&center=true&vCenter=true&width=320&lines=schwamm-builder"
alt="schwamm-builder"
/>
<br>
<img
src="https://placehold.co/400x400/000000/3B82F6?text=SCHWAMM%0ABUILDER"
width="500"
height="500"
alt="SCHWAMM BUILDER"
style="border-radius: 12px; margin-top: 15px;"
/>
</div>
<div  align="center">
<img
src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=30&pause=1500&color=EF4444&center=true&vCenter=true&width=150&lines=1.1.8%E2%80%8B"
alt="1.1.8"
/>
<br>
</div>

  

**Fluent message builders for WhatsApp** with [Baileys](https://github.com/WhiskeySockets/Baileys).

  

Build interactive buttons, lists, carousels, rich AI-style messages, payments, products, and media with a clean, chainable API.

  

```bash

npm  i  @whiskeysockets/baileys  schwamm-builder

```

  

---

  

## Changelog

### 1.1.9

- Fixed TableV2 rendering issue

### 1.1.8

- Fixed README

### 1.1.7


- added **`.xTabs`**  to **RichBuilder**
- **`xCarousel`** is now **`xActionCarousel`**
- `forwarded` now also have `forwardedScore`
- Fixed Builder Issues
- Fixed README

  

### 1.1.4 , 1.1.5 , 1.1.6

  

- Fixed readme errors

  

### 1.1.3

  

-  **`xOpts` expanded** — shared flags: `ai`, `secure`, `forward`, `status`, `bypassDownloadBtn` / `V2`, **`deleteForMe`**

- Removed standalone **`.xWAStatus()`** / **`.xForwarded()`** — use `.xOpts({ status: … })` and `.xOpts({ forward: n })`

-  **Other:** stickers + audio (PTT); basic media API only

-  **Rich:**  **`.xSocial(service, name, label?)`** — Instagram / Facebook / Threads / Telegram / website chips

- Status quote text is independent of message body (set via `status.text` / `status.message`)

  

### 1.1.2

  

- Fixed the **`Cannot find module './Selection'`** error on some terminals

  

### 1.1.1

  

- Added **`bypassDownloadBtnV2`** — more reliable download-button bypass when reopening a chat after WhatsApp was closed

- V2 supports optional `bypassDownloadInterval` and `bypassDownloadMax`

  

### 1.1.0

  

- Added **`bypassDownloadBtn`** (V1) via `.xOpts({ bypassDownloadBtn: true })`

- Fixed **`.xImage()`** (rich) — uses `GenAIImaginePrimitive` so banners render more reliably

  

---

  

## Table of Contents

  

- [Requirements](#requirements)

- [Quick Start](#quick-start)

- [Handling Button / List Clicks](#handling-button--list-clicks)

- [Builders Overview](#builders-overview)

- [Interactive Messages](#interactive-messages)

- [Rich Messages](#rich-messages)

- [Payment Messages](#payment-messages)

- [Other Messages](#other-messages)

- [Global Flags (xOpts)](#global-flags-xopts)

- [License](#license)

  

---

  

## Requirements

  

| **Node.js** | `18+` |
|---|---|
| **Peer dependency** | `@whiskeysockets/baileys` |
| **License** | `MIT` |

  

Your bot owns the WhatsApp socket. `schwamm-builder` only builds payloads and sends them through that socket.

  

⚠️ **Do not mix**  `interactive`, `rich`, and `payment` in a single message. Use **one** builder per `.send()`.

  

---

  

## Quick Start

  

```js

const  Builder = require('schwamm-builder');

  

// sock = Baileys socket

// jid = chat id string (or object with id / chat / remoteJid)

await  Builder.interactive()

.xText('Hello World')

.xButton('Click me', 'action_id')

.send(sock, jid);

```

  

---

  

## Handling Button / List Clicks

  

Clicks are **not** handled inside the builders. Wire this **once** in your message handler (`messages.upsert`), not in every command:

  

```js

const  Builder = require('schwamm-builder');

  

// inside messages.upsert, before normal text commands:

if (await  Builder.handleSelection(msg, sock, commands)) return;

  

// … then parse body / run command.run as usual

```

  

`handleSelection` reads the selected id from interactive / list / native-flow responses and calls `command.selected(sock, s, selectedId)` on every command that defines it.

  

### Low-level helper

  

```js

const { parseSelection } = require('schwamm-builder');

  

const  selectedId = parseSelection(msg);

if (selectedId) {

// route to your selected handlers

}

```

  

Typical sources (handled by `parseSelection`):

  

-  `interactiveResponseMessage`

-  `listResponseMessage`

-  `buttonsResponseMessage`

-  `nativeFlowResponseMessage` (`paramsJson` → `id`)

  

### Command with `selected`

  

```js

const  Builder = require('schwamm-builder');

  

module.exports = {

name:  'menu',

  

async  run(x, s) {

await Builder.interactive()

.xText('Choose')

.xButton('A', 'a')

.xButton('B', 'b')

.send(x, s.chat);

},

  

async  selected(x, s, selectedId) {

if (selectedId ===  'a') await s.m.reply('You picked A');

if (selectedId ===  'b') await s.m.reply('You picked B');

}

};

```

  

`x` / `sock` is whatever your bot passes as the Baileys socket. The builder does not care about the variable name.

  

---

  

## Builders Overview

  

| Builder | Purpose | Best for |
|---|---|---|
| `.interactive()` | Buttons, lists, carousels | User selections, menus |
| `.rich()` | Rich AI-style sections | Tables, code, media, social, HTML |
| `.payment()` | Payments, orders, products | Money flows |
| `.other()` | Text, image, video, sticker, audio | Simple media |

  

---

  

## Interactive Messages

  

Buttons, lists, nested lists, offers, and carousels for user interactions.

  

### Basic buttons

  

```js

await  Builder.interactive()

.xText('Choose an option')

.xFooter('My Bot')

.xButton('Option 1', 'opt1')

.xButton('Option 2', 'opt2')

.xButton('Review', 'review', 'REVIEW')

.send(sock, jid);

```

  

```js

.xButtons([

{ label:  'Yes', id:  'yes'  },

{ label:  'No', id:  'no'  },

{ label:  'Maybe', id:  'maybe', icon:  'REVIEW'  }

])

```

  

### Button types

  

| Method | Type | Use case |
|---|---|---|
| `.xButton(text, id, icon?)` | Quick reply | Simple choices |
| `.xUrlButton(text, url, { icon, useWebview })` | Open URL | External links |
| `.xCallButton(text, phone, icon?)` | Call | Phone numbers |
| `.xCopyButton(text, code, icon?)` | Copy | Promo codes |
| `.xReminder(text, id?, icon?)` | Reminder | Set reminders |

  

### Lists

  

```js

await  Builder.interactive()

.xText('Select a category')

.xFooter('Bot')

.xList('Categories', [

{

title:  'Accounts',

rows: [

{ title:  'Profile', id:  'profile', description:  'View your profile' },

{ title:  'Settings', id:  'settings', description:  'Manage settings' }

]

},

{

title:  'Support',

rows: [

{ title:  'Help', id:  'help', description:  'Get help' }

]

}

], 'REVIEW')

.send(sock, jid);

```

  

Shorthand:

  

```js

.xListRows('Select', [

{ title:  'Option A', id:  'a', description:  'First option'  },

{ title:  'Option B', id:  'b', description:  'Second option'  }

], 'Settings')

```

  

### Nested lists (bottom sheet)

  

```js

await  Builder.interactive()

.xText('Anti System Settings')

.xFooter('Bot')

.xNestedList('Configure', 'Anti')

.xList('Antilink', [{

title:  'Antilink',

rows: [

{ title:  'Enable', id:  'al_on' },

{ title:  'Disable', id:  'al_off' }

]

}], 'REVIEW')

.xList('Antispam', [{

title:  'Antispam',

rows: [

{ title:  'Enable', id:  'as_on' },

{ title:  'Disable', id:  'as_off' }

]

}], 'CALL')

.send(sock, jid);

```

  

### Offers

  

```js

.xOffer('Limited offer', {

url:  'https://example.com/pay',

code:  'SAVE20',

expiration: Math.floor(Date.now() /  1000) +  3600

})

```

  

### Carousels

  

```js

const  img = 'https://example.com/product.jpg';

  

await  Builder.interactive()

.xText('Browse products')

.xFooter('Shop')

.xCard({

title:  'Product 1',

text:  'Premium version',

image: img

})

.xButton('Buy', 'buy_1', 'REVIEW')

.xUrlButton('Details', 'https://example.com/product-1')

.xCard({

title:  'Product 2',

text:  'Standard version',

image: img

})

.xList('Options', [{

title:  'Version',

rows: [

{ title:  'Monthly', id:  'monthly' },

{ title:  'Yearly', id:  'yearly' }

]

}], 'REVIEW')

.send(sock, jid);

```

  

Or pass buttons in the card object, or use `.xCards([...])`.

  

### Nested carousel

  

```js

const  img = 'https://example.com/product.jpg';

  

await  Builder.interactive()

.xText('Settings & products')

.xFooter('Bot')

.xCard({

title:  'Account',

text:  'Manage your account',

image: img

})

.xNestedList('Options', 'Account Settings')

.xList('Profile', [{

title:  'Profile',

rows: [

{ title:  'Edit name', id:  'edit_name' },

{ title:  'Change email', id:  'change_email' }

]

}], 'REVIEW')

.xList('Privacy', [{

title:  'Privacy',

rows: [

{ title:  'Public', id:  'pub' },

{ title:  'Private', id:  'priv' }

]

}], 'CALL')

.xCard({

title:  'Support',

text:  'Get help',

image: img

})

.xList('Help', [{

title:  'Topics',

rows: [

{ title:  'FAQ', id:  'faq' },

{ title:  'Contact', id:  'contact' }

]

}], 'REVIEW')

.send(sock, jid);

```

  

---

  

## Rich Messages

  

Rich / GenAI-style messages with unified sections. Rendering is **client-dependent**.

  

### Text & headers

  

```js

await  Builder.rich()

.xHeader('Title')

.xBody('Body text with optional icon')

.xTip('Small metadata text inside message')

.xInfo('Disclaimer text shown under the message')

.send(sock, jid);

```

  

| Method | Purpose | Location |
|---|---|---|
| `.xHeader(text)` | Main heading | Inside message |
| `.xBody(text)` | Body content | Inside message |
| `.xTip(text)` | Small info text | Inside message (metadata style) |
| `.xInfo(text)` | Disclaimer/footer | **Under entire message** (`botMetadata`) |

  

### Media

  

Full banner image (`GenAIImaginePrimitive`) and small inline icons (LaTeX entities).

  

```js

.xImage('https://example.com/banner.jpg', {

mime:  'image/jpg',

width:  600,

height:  400,

status:  'READY',

update_text:  'done'

})

  

.xIcon('https://example.com/icon.png', {

width:  200,

height:  200,

fontHeight:  83.333,

padding:  10,

align:  'center',

expression:  'logo'

})

  

.xIcons([

'https://example.com/a.png',

'https://example.com/b.png'

], {

width:  150,

height:  150,

align:  'left'

})

```

  

| Method | Role | Size options |
|---|---|---|
| `.xImage(url, opts?)` | Large banner image | `width`, `height`, `mime`, `status`, `update_text` |
| `.xIcon(url, opts?)` | Small inline icon | `width`, `height`, `fontHeight`, `padding`, `align`, `expression` |
| `.xIcons(urls, opts?)` | Several icons | Same as `.xIcon` (shared opts) |

  

>  **Note:**  `width` / `height` are hints to the client. Prefer images already at the right resolution.

 

  

### Links

  

```js

.xLink('Open website', 'https://example.com')

```

  

### Copy

  

```js

.xCopy('SAVE20')

```

  

### Scroll text (horizontal cards)

  

```js

.xScrollText([

'Simple text card',

{ text:  'Card with content\nMultiline support'  },

{ text:  'Heading\nBody text', isHeading:  true  }

])

```

  you can use 
  ```js
  isHeading: true
  ```

### Tables

  

```js

.xTableV1('Users', [

['Name', 'Role', 'Status'],

['Alice', 'Admin', 'Active'],

['Bob', 'User', 'Inactive']

])

  

.xTableV2('Users', [

['Name', 'Role'],

['Alice', 'Admin'],

['Bob', 'User']

])

```
 you can use 
  ```js
  isHeading: true
  ```
  

### Code blocks

  

```js

.xCodeV1('const x = 42;', 'javascript')

.xCodeV2('console.log("hi")', 'python')

```

  

### Sources

  

```js

.xSources([

{ name:  'Wikipedia', url:  'https://en.wikipedia.org', subtitle:  'Web', favicon:  'https://example.com/favicon.ico'  },

])

```

  

### Citations

  

```js

.xCitations([

{ title:  'Article', url:  'https://example.com/article', snippet:  'Another insight'  }

])

```

  

### Suggestions

  

```js

.xSuggestions([

'Tell me more',

'Explain differently',

'Show examples'

])

```

  

### Posts

  

```js

.xPosts([{

username:  'john_doe',

title:  'Amazing sunset',

thumbnail_url:  'https://example.com/thumb.jpg',

post_url:  'https://example.com/post',

is_verified:  true,

likes_count:  1250,

comments_count:  45,

shares_count:  12

}])

```

  

### Reels

  

```js

.xReels([{

username:  'creator',

videoUrl:  'https://example.com/video.mp4',

thumbnailUrl:  'https://example.com/thumb.jpg',

is_verified:  true,

likes_count:  5000

}]),

```

  

### Products

  

```js

.xProducts([{

title:  'Premium Ebook',

brand:  'Author Name',

price:  '9.99 EUR',

product_url:  'https://example.com/shop',

image_url:  'https://example.com/product.jpg'

}])

```

  

### Action widgets

  

```js

.xActionRow('Quick actions', [

{ label:  'Start', toast:  'Starting...'  },

{ label:  'Cancel', toast:  'Cancelled'  }

])

  

.xActionCarousel([

{ title:  'Card 1', ctas: [{ label:  'Go', toast:  'Navigating...' }] },

{ title:  'Card 2', ctas: [{ label:  'Learn more' }] }

])

```

  

### HTML

  

Rich HTML (can include JS).

  

```js

const  html = `<html><body style="margin:0;padding:12px;font-family:sans-serif">

<h3>Hello</h3>

<p>Interactive HTML works here.</p>

</body></html>`;

  

await  Builder.rich()

.xHtml(html)

.send(sock, jid);

```

  

#### Sounds (local files only)

  

In HTML call `play('id')`.

  

```js

await  Builder.rich()

.xHtml(`<div>hi</div>`, {

sound:  './commands/mp/beep.mp3',

volume:  1,

loop:  false,

autoplay:  true

})

.send(sock, jid);

  

await  Builder.rich()

.xHtml(`

<button onclick="play('win')">Win</button>

<button onclick="play('lose')">Lose</button>

`, {

sounds: {

win:  './commands/mp/win.mp3',

lose:  './commands/mp/lose.mp3'

},

volume:  1,

loop:  false

})

.send(sock, jid);

```

  

| Option | Description |
|---|---|
| `sound` | Single file → ID `default` (`play('default')`) |
| `sounds` | Map of ID → local path |
| `volume` | Gain (default `1`) |
| `loop` | Default `false`; set `true` to loop |
| `autoplay` | `true` = first/default sound on open; or an ID string, e.g. `'win'` |

### Social profiles

  

```js

await  Builder.rich()

.xHeader('Lookup')

.xBody('Profil')

.xSocial('instagram', 'zuck')

.xSocial('facebook', 'zuck', 'See results')

.xSocial('threads', 'handle')

.xSocial('website', 'https://example.com', 'Open site')

.send(sock, jid);

```

  

| Arg | Description |
|---|---|
| `service` | `instagram` / `ig`, `facebook` / `fb`, `threads`, `telegram` / `tg`, `website` / `web` |
| `name` | Handle (without `@`) or full URL |
| `label` | Chip label (default: `See results`) |


### Tabs

Opens WhatsApp’s **See results** sheet with custom tabs. Only some types because the most arent working very well.

**Types:** `text` | `table` | `code` | `html`

```js
await Builder.rich()
  .xBody('Optional text above the sheet')
  .xTabs([
    { name: 'Info', type: 'text', content: 'Hello from a tab' },
    {
      name: 'Data',
      type: 'table',
      content: [
        ['Name', 'Value'],
        ['A', '1'],
        ['B', '2']
      ]
    },
    {
      name: 'Code',
      type: 'code',
      content: 'console.log("hi")',
      language: 'javascript'
    },
    {
      name: 'UI',
      type: 'html',
      content: '<div style="padding:12px">HTML tab</div>'
    }
  ])
  .send(sock, jid);
```

Multiple sections in one tab:

```js
.xTabs([
  {
    name: 'Mixed',
    sections: [
      { type: 'text', content: '**Intro**' },
      {
        type: 'table',
        content: [
          ['Cmd', 'Desc'],
          ['.ping', 'Latency']
        ]
      },
      {
        type: 'code',
        content: 'await Builder.rich().send(sock, jid)',
        language: 'javascript'
      }
    ]
  }
])
```

HTML tab with local sounds (same options as `.xHtml`):

```js
.xTabs([
  {
    name: 'Game',
    type: 'html',
    content: `
      <button onclick="play('spin')">Spin</button>
      <button onclick="play('win')">Win</button>
    `,
    options: {
      sounds: {
        spin: './commands/spin.mp3',
        win: './commands/win.mp3'
      },
      volume: 1
    }
  }
])
```


| Option       | Description                                      |
| ----------- | ------------------------------------------------ |
| `name`      | Tab title                                        |
| `type`      | `text` \| `table` \| `code` \| `html`            |
| `content`   | Text, table rows, code string, or HTML           |
| `language`  | Code language (default `javascript`)             |
| `sections`  | Multiple blocks in one tab (overrides `type`)    |
| `options`   | HTML only: `sound` / `sounds`, `volume`, `loop`, `autoplay` |
| `id`        | Optional; otherwise a UUID is assigned           |


  

### Complete example

  

```js

await  Builder.rich()

.xHeader('Schwamm Builder')

.xBody('Build WhatsApp messages with fluent APIs')

.xIcons(['https://example.com/a.png', 'https://example.com/b.png'], {

width:  300,

height:  300

})

.xScrollText([

{ text:  'Interactive\nButtons, lists, carousel', isHeading:  true  },

{ text:  'Rich\nTables, code, media, links', isHeading:  true  },

{ text:  'Payment\nRequests, orders, products', isHeading:  true  }

])

.xSocial('instagram', 'example')

.xLink('View docs', 'https://example.com/docs')

.xCopy('WELCOME2024')

.xSuggestions(['Show examples', 'How to install', 'Pricing'])

.xSources([

{ name:  'GitHub', url:  'https://example.com', favicon:  'https://example.com/favicon.ico'  }

])

.xTip('Limited time offer — valid until end of month')

.xInfo('Built by Schwamm')

.xOpts({ bypassDownloadBtn:  true  })

.send(sock, jid);

```

  

---

  

## Payment Messages

  

### Request payment

  

```js

await  Builder.payment()

.xRequest({

amount:  25,

currency:  'EUR',

text:  'Please pay 25 EUR for your order',

from:  'me@s.whatsapp.net'

})

.send(sock, jid);

```

  

### Order

  

```js

await  Builder.payment()

.xOrder({

text:  'Thanks for your order!',

amount:  67,

currency:  'EUR',

itemCount:  3,

thumbnail:  'https://example.com/thumb.png'

})

.send(sock, jid);

```

  

`thumbnail` can be a URL or Buffer.

  

### Product

  

```js

await  Builder.payment()

.xProduct({

title:  'Premium Plan',

description:  'Unlimited features for 1 month',

amount:  9.99,

currency:  'EUR',

image:  'https://example.com/product.png',

productId:  'sku-premium',

retailerId:  '',

businessOwnerJid:  'business@s.whatsapp.net'

})

.send(sock, jid);

```

  

`image` is required (URL or Buffer).

  

---

  

## Other Messages

  

```js

await  Builder.other()

.xText('Hello World')

.send(sock, jid);

  

await  Builder.other()

.xImage('https://example.com/pic.jpg', 'Look at this!')

.send(sock, jid);

  

await  Builder.other()

.xVideo('https://example.com/video.mp4', 'Check this out')

.send(sock, jid);

  

await  Builder.other()

.xSticker('https://example.com/sticker.webp')

.send(sock, jid);

  

await  Builder.other()

.xAudio('./audio.mp3')

.send(sock, jid);

  

await  Builder.other()

.xAudio('./voice.ogg', { ptt:  true  })

.send(sock, jid);

```

  

`xOpts` works on Other as well (AI, forward, status, `deleteForMe`, …).

  

---

  

## Global Flags (`xOpts`)

  

Available on **all** builders: `interactive`, `rich`, `payment`, `other`.

  

```js

.xOpts({

ai:  true,

secure:  true,

forward:  true,

forwardingCount: 271

status:  true,

// status: 'order',

// status: { type: 'order', text: 'Hi', itemCount: 1 },

// status: { type: 'product', title: 'Item', price: 9.99, currency: 'EUR' },

// status: { type: 'stickerpack', packName: 'My Pack' },

// status: { type: 'image' | 'video' | 'sticker' | 'text' },

bypassDownloadBtn:  true,

bypassDownloadBtnV2:  true,

bypassDownloadInterval:  2000,

bypassDownloadMax:  0,

deleteForMe:  true

})

```

  

| Opt | Effect |
|---|---|
| `ai` | AI/bot label — **private chats only** |
| `secure` | Secure meta / business attributes |
| `forward` | `number` or `true` → forwarded score; `false` / `null` to clear |
| `status` | `true` \| type string \| `{ type, text, message, itemCount, title, price, currency, packName }` |
| `bypassDownloadBtn` | **V1** — rich inline render instead of download button |
| `bypassDownloadBtnV2` | **V2** — more reliable after closing WhatsApp and reopening the chat |
| `bypassDownloadInterval` | Delay in ms between V2 attempts |
| `bypassDownloadMax` | Max V2 attempts (`0` = unlimited / continuous) |
| `deleteForMe` | After send, try **delete for me only** (others still see it) |

  

### Status quote

  

```js

await  Builder.other()

.xText('Visible chat text')

.xOpts({

status: { type:  'order', text:  'Order line in quote', itemCount:  2 }

})

.send(sock, jid);

```

  

### Forwarded

  

```js

await  Builder.interactive()

.xText('Forwarded message')

.xButton('OK', 'ok')

.xOpts({ forward:  true, forwardingCount: 67 })

.send(sock, jid);

```

  

### AI label

  

```js

await  Builder.rich()

.xBody('This is an AI response')

.xOpts({ ai:  true  })

.send(sock, jid);

```

  

⚠️ Only works in private chats.

  

### Bypass download (rich)

  

**V1**

  

```js

await  Builder.rich()

.xImage('https://example.com/banner.jpg', { mime:  'image/jpg'  })

.xBody('Banner test')

.xOpts({ bypassDownloadBtn:  true  })

.send(sock, jid);

```

  

**V2**

  

```js

await  Builder.rich()

.xImage('https://example.com/banner.jpg', { mime:  'image/jpg'  })

.xBody('Banner test')

.xOpts({

bypassDownloadBtnV2:  true,

bypassDownloadInterval:  2000,

bypassDownloadMax:  0

})

.send(sock, jid);

```

  

### Delete for me only

  

```js

await  Builder.other()

.xText('Hallo')

.xOpts({ deleteForMe:  true  })

.send(sock, jid);

```

  

### Combined flags

  

```js

await  Builder.other()

.xText('Hi')

.xOpts({

ai:  true,

secure:  true,

forward:  3,

status: { type:  'text' },

deleteForMe:  false

})

.send(sock, jid);

```

  

---

  

<p  align="center">
<strong>MADE BY SCHWAMM</strong>
</p>

  

## License

  

MIT