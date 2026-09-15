<div align="center">
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
<div align="center">
  <img
    src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=30&pause=1500&color=EF4444&center=true&vCenter=true&width=150&lines=1.1.2%E2%80%8B"
    alt="1.1.2"
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

  

### 1.1.2

- Fixed the **`Cannot find module './Selection'`** Error occured on some terminals

  

### 1.1.1

- Added **`bypassDownloadBtnV2`** — more reliable download-button bypass when reopening a chat after WhatsApp was closed (V1 often did not auto-download again in that case)

- V2 supports optional `bypassDownloadInterval` and `bypassDownloadMax`

  

### 1.1.0

- Added **`bypassDownloadBtn`** (V1) via `.xOpts({ bypassDownloadBtn: true })`

- Fixed **`.xImage()`** — uses `GenAIImaginePrimitive` so banners render reliably

  

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

- [Global Flags](#global-flags)

- [Tips & Tricks](#tips--tricks)

- [License](#license)

  

---

  

## Requirements

  
| | |
|---|---|
| **Node.js** | 18+ |
| **Peer dependency** | `@whiskeysockets/baileys` (install in your bot) |
| **License** | MIT |

  

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

  

If you build `s` yourself:

  

```js

const { parseSelection } = require('schwamm-builder');

  

const  selectedId = parseSelection(msg);

if (selectedId) {

// route to your selected handlers

}

```

  

Typical sources (handled for you by `parseSelection`):

  

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
|---------|---------|----------|
| `.interactive()` | Buttons, lists, carousels | User selections, menus |
| `.rich()` | Rich AI-style sections | Tables, code, media, scroll text |
| `.payment()` | Payments, orders, products | Money flows |
| `.other()` | Text, images, videos, stickers | Simple media |

  

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

.xButton('Review', 'review', 'REVIEW') // with icon

.send(sock, jid);

```

  

Or add multiple buttons at once:

  

```js

.xButtons([

{ label:  'Yes', id:  'yes'  },

{ label:  'No', id:  'no'  },

{ label:  'Maybe', id:  'maybe', icon:  'REVIEW'  }

])

```

  

### Button types

  

| Method | Type | Use case |
|--------|------|----------|
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

  

### Image headers

  

```js

.xImage('https://example.com/banner.jpg')

.xTitle('Welcome')

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
|--------|---------|----------|
| `.xHeader(text)` | Main heading | Inside message |
| `.xBody(text)` | Body content | Inside message |
| `.xTip(text)` | Small info text | Inside message (metadata style) |
| `.xInfo(text)` | Disclaimer/footer | **Under entire message** (botMetadata) |

  

### Media

  

Full banner image (`GenAIImaginePrimitive`) and small inline icons (LaTeX entities).

  

```js

// Full banner / image — size is optional

.xImage('https://example.com/banner.jpg', {

mime:  'image/jpg', // default: 'image/png'

width:  600,

height:  400,

status:  'READY',

update_text:  'done'  // optional

})

  

// Single inline icon — transparent PNGs work best

.xIcon('https://example.com/icon.png', {

width:  200, // default: 120

height:  200, // default: 120

fontHeight:  83.333, // vertical space in text flow

padding:  10,

align:  'center', // 'left' | 'center'

expression:  'logo'  // placeholder label inside the entity

})

  

// Multiple icons in one row

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
|--------|------|--------------|
| `.xImage(url, opts?)` | Large banner image | `width`, `height`, `mime`, `status`, `update_text` |
| `.xIcon(url, opts?)` | Small inline icon | `width`, `height`, `fontHeight`, `padding`, `align`, `expression` |
| `.xIcons(urls, opts?)` | Several icons | same as `.xIcon` (shared opts) |

  

>  **Note:**  `width` / `height` are hints to the client — not guaranteed CSS-like sizing. For exact look, serve an image that is already the right resolution.

  

### Links & copy

  

```js

.xLink('Open website', 'https://example.com')

.xLink('View docs', 'https://docs.example.com')

  

.xCopy('SAVE20')

.xCopy('welcome-code-2024')

```

  

### Scroll text (horizontal cards)

  

```js

.xScrollText([

'Simple text card',

{ text:  'Card with content\nMultiline support'  },

{ text:  'Heading\nBody text', isHeading:  true  }

])

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

  

### Code blocks

  

```js

.xCodeV1('const x = 42;', 'javascript')

.xCodeV2('console.log("hi")', 'python')

```

  

### Sources & citations

  

```js

.xSources([

{ name:  'Wikipedia', url:  'https://en.wikipedia.org', subtitle:  'Web', favicon:  'https://example.com/favicon.ico'  },

{ name:  'GitHub', url:  'https://github.com', subtitle:  'Code', favicon:  'https://example.com/favicon.ico'  }

])

  

// shorthand tuples: [favicon, url, name, subtitle?]

.xSources([

['https://example.com/favicon.ico', 'https://example.com', 'Example Site', 'Documentation']

])

```

  

```js

.xBody('This is true according to science.')

.xCitations([

{ title:  'Study Title', url:  'https://example.com/study', snippet:  'Key finding here'  },

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

  

### Posts, reels, products

 

  

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

  

.xReels([{

username:  'creator',

videoUrl:  'https://example.com/video.mp4',

thumbnailUrl:  'https://example.com/thumb.jpg',

is_verified:  true,

likes_count:  5000

}])

  

.xProduct({

title:  'Premium Ebook',

brand:  'Author Name',

price:  '9.99 EUR',

product_url:  'https://example.com/shop',

image_url:  'https://example.com/product.jpg'

})

  

.xProducts([/* array of products */])

```

  

### Action widgets

  

  

```js

.xActionRow('Quick actions', [

{ label:  'Start', toast:  'Starting...'  },

{ label:  'Cancel', toast:  'Cancelled'  }

])

  

.xCarousel([

{ title:  'Card 1', ctas: [{ label:  'Go', toast:  'Navigating...' }] },

{ title:  'Card 2', ctas: [{ label:  'Learn more' }] }

])

```

  

### HTML

Rich HTML (also can render javascript)

  

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
|--------|-------------|
| `sound` | Single file → id `default` (`play('default')`) |
| `sounds` | Map of id → local path |
| `volume` | Gain (default `1`) |
| `loop` | Default `false`; set `true` to loop |
| `autoplay` | `true` = first/default sound on open; or id string e.g. `'win'` |

  

  

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

.xLink('View docs', 'https://example.com/docs')

.xLink('Try it now', 'https://example.com')

.xCopy('WELCOME2024')

.xSuggestions(['Show examples', 'How to install', 'Pricing'])

.xSources([

{ name:  'GitHub', url:  'https://example.com', favicon:  'https://example.com/favicon.ico'  }

])

.xTip('Limited time offer — valid until end of month')

.xInfo('Built by Schwamm')

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

```

  

---

  

## Global Flags

  

Available on **all** builders: `interactive`, `rich`, `payment`, `other`.

  

### Shared options via `.xOpts()`

  

```js

.xOpts({

ai:  true, // AI / bot label (private chats only)

secure:  true, // secure meta / business attributes

bypassDownloadBtn:  true, // V1 — rich download-button bypass

bypassDownloadBtnV2:  true  // V2 — stronger bypass (see below)

})

```

  

| Opt | Effect |
|-----|--------|
| `ai` | AI/bot label — **private chats only** (`@s.whatsapp.net` / `@lid`) |
| `secure` | Secure meta / business attributes |
| `bypassDownloadBtn` | **V1** — forces rich content to render inline (edit trick) instead of a download button |
| `bypassDownloadBtnV2` | **V2** — same goal as V1, but more reliable after **closing WhatsApp and reopening the chat** (V1 often did not auto-download / re-render in that case) |

  



  

**V1 example**

  

```js

await  Builder.rich()

.xImage('https://example.com/banner.jpg', { mime:  'image/jpg'  })

.xBody('Banner test')

.xOpts({ bypassDownloadBtn:  true  })

.send(sock, jid);

```

  

**V2 example** (with optional timing / retry controls)

  

```js

await  Builder.rich()

.xImage('https://example.com/banner.jpg', { mime:  'image/jpg'  })

.xBody('Banner test')

.xOpts({

bypassDownloadBtnV2:  true,

bypassDownloadInterval:  2000, // ms between bypass attempts

bypassDownloadMax:  0  // 0 = unlimited; or set a max count

})

.send(sock, jid);

```

  

| V2 option | Description |
|-----------|-------------|
| `bypassDownloadBtnV2` | Enable V2 bypass |
| `bypassDownloadInterval` | Delay in ms between bypass attempts |
| `bypassDownloadMax` | Max attempts (`0` = unlimited / continuous) |

  

### AI label

  

```js

await  Builder.rich()

.xBody('This is an AI response')

.xOpts({ ai:  true  })

.send(sock, jid);

```

  

⚠️ Only works in private chats

  

### Forwarded

  

```js

await  Builder.interactive()

.xText('Forwarded message')

.xButton('OK', 'ok')

.xForwarded(1) // score 1–7

.send(sock, jid);

```

  

### Status quotes

  

```js

.xWAStatus('Nice!', 'text')

.xWAStatus('Great photo', 'image')

.xWAStatus('Cool video', 'video')

.xWAStatus('Love this', 'stickerpack', 'Pack Name')

.xWAStatus('Interested!', 'product', {

title:  'Premium Plan',

price:  9.99,

currency:  'EUR'

})

.xWAStatus('Thanks!', 'order', {

message:  'Processing your order',

itemCount:  2

})

```

  

### Combined flags

  

```js

await  Builder.other()

.xText('Hi')

.xForwarded(3)

.xOpts({ ai:  true, secure:  true  })

.send(sock, jid);

```

---

<p align="center">
  <strong>MADE BY SCHWAMM</strong>
</p>


  

## License

  

MIT