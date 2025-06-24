# Nanobrowser UI and Configuration Components

This document outlines the user-facing components of the Nanobrowser extension, including the main interface, settings page, and permission-handling pages.

## 1. Main User Interface: The Side Panel

This is the primary interface where users interact with the agent to issue commands and view progress.

*   **`side-panel/index.html`**: The main HTML file that sets up the structure for the side panel.
*   **`side-panel/assets/index-9rsyy5zg.js`**: This is the compiled and minified JavaScript containing all application logic for the side panel. It is likely built using a framework like React and handles everything from rendering chat messages to communicating with the background script.
*   **`side-panel/icons/*.svg`**: This folder contains the SVG icons (navigator, planner, validator, etc.) displayed in the side panel UI to represent the different AI agent roles.

## 2. Extension Settings: The Options Page

This is where users can configure API keys, select AI/LLM models, and adjust the agent's behavior.

*   **`options/index.html`**: The HTML file for the options page.
*   **`options/assets/index-BIGoF_ZM.js`**: The compiled JavaScript that powers the options page UI. It is responsible for handling the forms, managing state, and saving settings to `chrome.storage`.

## 3. Permission-Request Page

A dedicated page to handle specific browser permission requests (e.g., microphone access) in a user-friendly way.

*   **`permission/index.html`**: The simple HTML structure for the permission request pop-up window.
*   **`permission/permission.js`**: The JavaScript logic that handles the permission request flow, updates the status text on the page, and closes the window upon success.