# Nanobrowser-v0.1.7 Extension Architecture

Nanobrowser appears to be an AI-powered Chrome extension designed to automate web tasks. It leverages various Chrome APIs and a modular architecture to achieve this.

## 1. `buildDomTree.js`

This JavaScript file is a core component injected into web pages to provide the AI agent with a structured understanding of the page's content and interactive elements.

### Purpose

To traverse the web page's Document Object Model (DOM) and extract information about visible, interactive elements, including their attributes, XPath, and position. It also handles visually highlighting these elements.

### Key Functionality

*   **`window.buildDomTree(args)`**: The main entry point. It initiates the DOM traversal and processing.

*   **DOM Traversal**: Recursively walks through the DOM, including elements within iframes and shadow DOM.

*   **Element Filtering**: It intelligently skips non-relevant elements (like `script`, `style`, `meta`, `noscript`) and the extension's own highlighting elements to focus on meaningful content.

*   **Visibility Checks (`isElementVisible`, `isTextNodeVisible`, `isInExpandedViewport`)**: Determines if elements and text nodes are actually visible to the user, considering CSS properties (`display`, `visibility`, `opacity`) and their bounding boxes within the viewport (with optional expansion).

*   **Interactivity Heuristics (`isInteractiveElement`, `isElementDistinctInteraction`, `isHeuristicallyInteractive`)**: This is a sophisticated part. It identifies elements that users can interact with (buttons, inputs, links, etc.) by checking:
    *   Standard HTML tags (`a`, `button`, `input`, `select`, `textarea`).
    *   CSS `cursor` styles (e.g., `pointer`).
    *   ARIA roles (e.g., `role="button"`).
    *   `contenteditable` attributes.
    *   Presence of event listeners (`onclick`, `onmousedown`, etc.).
    *   It also tries to distinguish between an interactive element that's part of a larger interactive component (e.g., a `<span>` inside a `button`) and one that represents a distinct, clickable action.

*   **XPath Generation (`getXPathTree`)**: Creates a simplified XPath for each element, useful for locating elements later. It uses a `WeakMap` for caching XPath strings to improve performance.

*   **Highlighting (`highlightElement`)**: If `showHighlightElements` is true, it creates visual overlays and labels (with the element's index) on top of interactive elements. This helps users (and potentially visual AI models) understand what the agent "sees" as interactable.

*   **`DOM_HASH_MAP`**: Stores a flat map of all processed DOM nodes, indexed by a unique ID. This allows the background script to reference specific elements without direct DOM access.

*   **Performance Metrics & Caching**: When `debugMode` is enabled, it tracks detailed performance metrics for DOM operations (like `getBoundingClientRect`, `getComputedStyle`) and implements caching (`DOM_CACHE` using `WeakMap`) to optimize repeated DOM queries.

## 2. Nanobrowser-v0.1.7 Folder Structure and Overall Explanation

The `nanobrowser-v0.1.7` directory contains the compiled code for a Chrome extension. It follows a common architecture for browser automation tools.

### `manifest.json`

This is the blueprint of the Chrome extension. It declares its name, version, description, and crucial permissions (`storage`, `scripting`, `tabs`, `debugger`, `sidePanel`).
*   It specifies `background.iife.js` as the service worker (running in the background), `options/index.html` as the settings page, and `side-panel/index.html` as the side panel UI.
*   `host_permissions` and `content_scripts` allow it to inject code and interact with web pages.
*   `debugger` permission is key for advanced browser control.

### `background.iife.js`

This is the Service Worker, the central brain of the extension. It runs in the background and orchestrates all major operations.
*   **Browser Automation Engine**: It integrates with Puppeteer/WebDriver BiDi (via polyfills and the `sB` module) to control browser tabs, navigate, click, type, and gather detailed page information.
*   **Storage Management**: Uses `chrome.storage.local` and `chrome.storage.session` to store:
    *   LLM API Keys: Credentials for various AI models (OpenAI, Anthropic, Gemini, Groq, Ollama, Azure OpenAI, OpenRouter, Cerebras).
    *   Agent Models: Configurations (model name, parameters) for different AI roles (Planner, Navigator, Validator).
    *   General Settings: Global settings like max steps, failure tolerance, use of vision models, highlighting, and page load timings.
    *   Firewall Settings: Allow/deny lists for URLs to control agent navigation.
    *   Speech-to-Text Model: Configuration for voice input.
*   **AI Agent Orchestration**: It contains the `Executor` class, which manages the lifecycle of AI tasks. The `Executor` coordinates three main AI agents:
    *   **Planner**: Decides the high-level steps to achieve a task.
    *   **Navigator**: Executes browser actions (clicks, types, navigates) based on the plan.
    *   **Validator**: Checks if the task has been successfully completed.
*   **Dynamic Script Injection**: It listens for tab updates and injects `buildDomTree.js` into newly loaded `http(s)` pages, ensuring the agent always has access to the DOM information.
*   **Communication Hub**: It acts as a message broker, receiving commands from the side panel and sending back execution updates.

### `content/index.iife.js`

A simple Content Script that runs within the context of web pages.
*   Its primary purpose is to serve as an entry point for the background script to inject further scripts (like `buildDomTree.js`) into the page's isolated world, allowing them to interact directly with the page's DOM.

### `options/` folder

Contains the Options Page (`index.html` and compiled React assets).
*   This UI allows users to configure all the settings stored in `chrome.storage.local`, including LLM providers, agent models, general behavior, and firewall rules.

### `permission/` folder

Contains a dedicated Permission Request Page (`index.html` and `permission.js`).
*   This is used to explicitly ask the user for permissions like microphone access (for speech-to-text) in a separate window, ensuring transparency and user control.

### `side-panel/` folder

Contains the Side Panel UI (`index.html` and compiled React assets).
*   This is the main interactive interface for the user. It provides a chat-like experience where users can input tasks, view the agent's progress, and manage chat sessions.
*   It communicates with `background.iife.js` to send tasks, receive real-time updates on agent actions and thoughts, and manage chat history.
*   It integrates features like speech-to-text input.
*   The `icons/` subfolder contains SVG icons used in the side panel to represent different AI roles.

## Overall Architecture Summary

The Nanobrowser extension operates by having a central Service Worker (`background.iife.js`) that orchestrates AI agents and browser automation. It injects a specialized script (`buildDomTree.js`) into web pages via a Content Script to gather detailed DOM information. The Side Panel provides the user interface for interaction, sending commands to the Service Worker and receiving real-time updates. The Options Page and Permission Page handle configuration and permission requests, respectively. This modular design allows for powerful, AI-driven web automation capabilities.