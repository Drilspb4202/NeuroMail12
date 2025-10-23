# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Development Commands

### Running the Application

**Python HTTP Server (Recommended):**
```bash
python -m http.server 8000
```
The application will be available at `http://localhost:8000`

**PowerShell Server (Windows):**
```powershell
powershell -ExecutionPolicy Bypass -File start-server.ps1
```
This starts a custom PowerShell-based HTTP server on port 8090

**Batch Script (Windows):**
```cmd
run-with-python.bat
```
Simple wrapper that runs the Python HTTP server on port 8000

### Testing and Debugging

**Browser Console Testing:**
```javascript
// Check API connection status
window.mailSlurpApi.checkConnection().then(status => console.log(status));

// Test API key pool status
window.mailSlurpApi.keyPool.getPoolStatus();

// Clear local storage for fresh start
localStorage.clear();
```

**Manual API Key Testing:**
Open browser console and check current API keys being used:
```javascript
console.log("Current API Key:", window.mailSlurpApi.apiKey.substring(0, 8) + "...");
console.log("API Mode:", window.mailSlurpApi.apiMode);
```

## Architecture Overview

### Core Application Structure

**Main Application Classes:**
- `MailSlurpApp` (`js/app.js`) - Central application controller that orchestrates API and UI interactions
- `MailSlurpApi` (`js/api.js`) - Handles all MailSlurp API communication with automatic retry logic
- `MailSlurpUI` (`js/ui.js`) - Manages UI components, modals, navigation, and user interactions

**API Key Management System:**
- `ApiKeyPool` (`js/api-key-pool.js`) - Manages pool of public API keys with automatic rotation when limits are reached
- `ApiKeyManager` (`js/apiKeyManager.js`) - Handles user API key lifecycle and monetization features
- Supports three API modes: public, personal, and combined for seamless fallback

**Supporting Modules:**
- `DataGenerator` (`js/generator.js`) - Generates fake user data for testing/registration purposes
- `I18nManager` (`js/i18n.js`) - Internationalization support for Russian and English
- `PerformanceOptimizer` (`js/performance-optimizer.js`) - Optimizes scrolling and rendering performance

### Key Technical Patterns

**API Resilience:**
- Automatic API key rotation when limits are exceeded
- Exponential backoff retry mechanism for failed requests
- Graceful degradation between public and personal API keys
- Connection status monitoring with user notifications

**State Management:**
- LocalStorage for persistent settings and current inbox state
- Event-driven architecture for cross-component communication
- Automatic cleanup of expired data and invalid API keys

**Mobile-First Design:**
- Responsive navigation with separate mobile menu
- Touch-optimized UI components
- Performance optimizations for mobile scrolling

## Working with API Keys

### Public API Key Pool

The application uses a sophisticated API key pool system located in `js/api-key-pool.js`. When working with API keys:

**Adding New Keys:**
```javascript
// Update the publicKeys array in ApiKeyPool constructor
this.publicKeys = [
    {
        key: 'new_api_key_here',
        usageCount: 0,
        lastUsed: null,
        isExhausted: false,
        monthlyReset: new Date().getTime()
    }
];
```

**Testing Key Validity:**
```javascript
// Check if current keys are working
const isUsingOldKey = window.mailSlurpApi.isUsingDeprecatedKey();
if (isUsingOldKey) {
    await window.mailSlurpApi.switchToWorkingApiKey();
}
```

### Personal API Keys

Users can input personal MailSlurp API keys in Settings. The system automatically switches between public pool and personal keys based on availability and user preference.

## Email Management Architecture

### Inbox Lifecycle

**Creation Flow:**
1. User clicks "Create Inbox" → Opens modal
2. `MailSlurpApp.createInbox()` calls API with rate limiting checks
3. New inbox appears in table with copy-to-clipboard functionality
4. Auto-deletion timer starts (5 minutes for public API)

**Email Handling:**
1. Emails are fetched via `MailSlurpApi.getEmails(inboxId)`
2. Email viewer supports HTML/Markdown rendering with sanitization
3. Real-time email checking via interval-based polling
4. Unread email counter updates across desktop and mobile UI

### Multi-Language Support

The `I18nManager` handles dynamic text translation:

```javascript
// Getting localized text
const message = window.i18n.t('inbox_loading'); // Returns: "Загрузка почтовых ящиков..."

// Switching languages
window.i18n.setLanguage('en'); // Changes UI to English
```

## Development Guidelines

### Code Organization

- **Class-based architecture**: Each major feature is a ES6 class with clear separation of concerns
- **Event-driven communication**: Use CustomEvents for cross-component messaging
- **Defensive programming**: Always check for element existence before DOM manipulation
- **LocalStorage management**: Use consistent key naming and clean up expired data

### Performance Considerations

The `PerformanceOptimizer` class handles:
- Throttled scroll events to prevent UI jank
- Lazy loading of images in email content
- DOM update batching during rapid user interactions
- Mobile-specific optimizations for smooth scrolling

### Testing Workflow

**Local Development Testing:**
1. Start server using Python HTTP server
2. Open browser console to monitor API calls and errors
3. Test with different API keys by switching modes in Settings
4. Use the data generator to create test accounts
5. Verify mobile responsiveness by resizing window

**API Key Rotation Testing:**
```javascript
// Force exhaust current key for testing rotation
window.mailSlurpApi.keyPool.markCurrentKeyExhausted();
// Next API call should automatically switch to new key
```

## Common Development Tasks

### Adding New UI Sections

1. Add HTML structure to `index.html`
2. Add navigation item with `data-target` attribute
3. Implement section logic in `MailSlurpApp` or `MailSlurpUI`
4. Add translations to `I18nManager.translations` objects
5. Test responsive design on mobile

### Implementing New API Endpoints

1. Add method to `MailSlurpApi` class
2. Include proper error handling and retry logic
3. Update API key usage tracking if needed
4. Add UI integration in `MailSlurpApp`

### Updating Translations

1. Edit both `ru` and `en` objects in `js/i18n.js`
2. Use `data-i18n` attributes in HTML for automatic updates
3. Test language switching functionality

## Important Configuration

### API Rate Limits

The application handles MailSlurp API limits automatically:
- Monthly inbox creation limit: 50 per key
- Daily request limit: 250 per key
- Automatic key rotation when limits are reached

### Security Considerations

- API keys are stored in localStorage (consider encryption for production)
- Email content is sanitized before display
- No sensitive data is logged to console in production builds
- CORS restrictions apply to API calls

### Mobile Optimizations

- Fixed mobile navigation at bottom of screen
- Hide/show navigation on scroll for better UX
- Touch-optimized button sizes and spacing
- Responsive table layouts for email lists