---
title: "Anki Translator Chrome Extension"
description: "A Chrome extension for translating Anki flashcards using GPT-4"
date: 2025-10-09T10:00:00+03:00
tags: ["typescript", "chrome-extension", "anki", "deepl", "language-learning", "react"]
categories: ["projects", "development"]
description: "The story of creating a Chrome extension for automatic Anki flashcard creation with DeepL API integration"
featured: true
draft: false
ShowToc: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
---

## The Problem That Started It All

One month before my Swedish language exam. What does a normal person do? Studies the language. What do I do? Write a Chrome extension! 🤦‍♂️

But seriously, the problem was real. While learning Swedish, I constantly encountered unfamiliar words in articles, documentation, and social media. The standard process looked like this:

1. Select a word
2. Open DeepL in a new tab
3. Paste the text
4. Get the translation
5. Open Anki
6. Create a new card
7. Copy the original and translation
8. Save

By the time I finished this entire cycle, I had already forgotten the context in which I encountered the word. Classic context switching problem.

Meet [**Anki Translator**](https://chromewebstore.google.com/detail/hkaijlopnccgeebfkdbkkekdpkekapco?utm_source=item-share-cb) — my solution to efficient language learning.

## Technical Solution

The idea was simple: **one click → ready flashcard in Anki**. But the implementation turned out to be more interesting than it seemed.

### Extension Architecture

Chrome Extensions have a specific architecture with several isolated contexts:

```typescript
// Manifest V3 structure
{
  "manifest_version": 3,
  "background": {
    "service_worker": "background.js"  // Background process
  },
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["content.js"]              // Page script
  }],
  "action": {
    "default_popup": "popup.html"     // Extension UI
  }
}
```

**Background Script** — central coordinator, handles API requests and inter-component communication.

**Content Script** — works in the web page context, tracks text selection and shows context menu.

**Popup** — React application for settings and manual translation input.

### Text Selection Handling

The most important part — intercepting user text selection:

```typescript
// content/content.ts
let selectedText = "";

document.addEventListener("mouseup", () => {
  const selection = window.getSelection();
  if (selection && selection.toString().trim()) {
    selectedText = selection.toString().trim();
    // Update context menu
    chrome.runtime.sendMessage({
      action: "updateContextMenu",
      hasSelection: true,
    });
  }
});

// Context menu handling
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "getSelectedText") {
    sendResponse({ text: selectedText });
  }
});
```

### DeepL API Integration

DeepL provides an excellent REST API with support for 30+ languages:

```typescript
// background/deepl.ts
class DeepLTranslator {
  private apiKey: string;
  private baseUrl: string;

  constructor(apiKey: string, isPro: boolean) {
    this.apiKey = apiKey;
    this.baseUrl = isPro
      ? "https://api.deepl.com/v2"
      : "https://api-free.deepl.com/v2";
  }

  async translate(text: string, sourceLang: string, targetLang: string) {
    const response = await fetch(`${this.baseUrl}/translate`, {
      method: "POST",
      headers: {
        Authorization: `DeepL-Auth-Key ${this.apiKey}`,
        "Content-Type": "application/x-www-form-urlencoded",
      },
      body: new URLSearchParams({
        text,
        source_lang: sourceLang,
        target_lang: targetLang,
      }),
    });

    const data = await response.json();
    return data.translations[0].text;
  }
}
```

### Anki Integration via AnkiConnect

AnkiConnect is an Anki plugin that provides an HTTP API for external applications:

```typescript
// background/anki.ts
class AnkiConnector {
  private readonly ankiConnectUrl = "http://localhost:8765";

  async addNote(front: string, back: string, deck: string, tags: string[]) {
    const note = {
      deckName: deck,
      modelName: "Basic",
      fields: {
        Front: front,
        Back: back,
      },
      tags,
    };

    const response = await fetch(this.ankiConnectUrl, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        action: "addNote",
        version: 6,
        params: { note },
      }),
    });

    const result = await response.json();
    if (result.error) {
      throw new Error(result.error);
    }

    return result.result; // Created card ID
  }
}
```

### React UI with TypeScript

For the interface, I chose React + TypeScript + TailwindCSS. The popup needs to be fast and intuitive:

```typescript
// popup/TranslationPopup.tsx
interface TranslationState {
  originalText: string;
  translatedText: string;
  isLoading: boolean;
  error: string | null;
}

const TranslationPopup: React.FC = () => {
  const [state, setState] = useState<TranslationState>({
    originalText: "",
    translatedText: "",
    isLoading: false,
    error: null,
  });

  const handleTranslate = async () => {
    setState((prev) => ({ ...prev, isLoading: true, error: null }));

    try {
      const result = await chrome.runtime.sendMessage({
        action: "translate",
        text: state.originalText,
      });

      setState((prev) => ({
        ...prev,
        translatedText: result.translation,
        isLoading: false,
      }));
    } catch (error) {
      setState((prev) => ({
        ...prev,
        error: error.message,
        isLoading: false,
      }));
    }
  };

  // JSX with TailwindCSS classes...
};
```

### Webpack Build Configuration

Chrome Extensions require special Webpack configuration:

```javascript
// webpack.config.js
module.exports = {
  entry: {
    background: "./src/background/background.ts",
    content: "./src/content/content.ts",
    popup: "./src/popup/index.tsx",
    options: "./src/options/index.tsx",
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].js",
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader", "postcss-loader"],
      },
    ],
  },
  plugins: [
    new CopyWebpackPlugin({
      patterns: [{ from: "manifest.json" }, { from: "public" }],
    }),
  ],
};
```

## Privacy and Security

One of the key principles is **zero data collection**. The extension:

- Does not collect any user data
- API keys are stored locally in the browser
- Translations are sent directly from browser to DeepL
- No external servers or analytics

All data remains on the user's device.

## Results

After one month of active use:

- Created 800+ cards without extra clicks
- Time to create one card: 15 seconds → 3 seconds
- Learned enough Swedish to pass the exam! 🎉

## Tech Stack

- **TypeScript** — for type safety and better DX
- **React** — for UI components
- **TailwindCSS** — for rapid styling
- **Webpack** — for building and bundling
- **Chrome Extensions API** — for browser integration
- **DeepL API** — for quality translations
- **AnkiConnect** — for Anki integration

## Conclusions

Sometimes the best projects are born from personal pain. When existing tools don't solve your problem efficiently — it's time to write your own.

The project turned out to be not only useful for language learning, but also excellent practice for working with:

- Chrome Extensions API
- Inter-component communication in browsers
- External API integration
- React in the constrained context of extensions

And most importantly — it actually works and saves time every day!

**🔗 Check out the project:** [Anki Translator](https://chromewebstore.google.com/detail/hkaijlopnccgeebfkdbkkekdpkekapco?utm_source=item-share-cb)
