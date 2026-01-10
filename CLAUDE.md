# CLAUDE.md - Divine Node Code Academy

**CRITICAL: Read `/home/gh0st/dvn/ARCHITECTURE_STANDARDS.md` FIRST before making ANY changes.**

## Project Overview

Divine Node Code Academy is an interactive web-based coding education platform for complete beginners. It teaches HTML, CSS, JavaScript, and development workflows through hands-on lessons with live code validation and visual feedback.

## Architecture Status

⚠️ **NEEDS REFACTORING** - This project was initially built with monolithic architecture and needs to be restructured following `/home/gh0st/dvn/ARCHITECTURE_STANDARDS.md`.

### Current Issues:
1. ❌ `js/tutorial-engine.js` is 1116 lines (should be <200 lines)
2. ❌ Mixed concerns (data loading + rendering + validation in one file)
3. ❌ No component isolation
4. ❌ No automated testing
5. ❌ No build tooling
6. ❌ JSON data wasn't being loaded (was using embedded data)

### Refactoring Plan:

**Phase 1: Modular Structure** (DO THIS FIRST)
```
code-academy/
├── index.html
├── package.json                # NEW - npm dependencies
├── vite.config.js             # NEW - Vite configuration
├── docs/
│   ├── README.md
│   └── ARCHITECTURE.md
├── src/                       # NEW - Source code
│   ├── core/
│   │   ├── TutorialEngine.js      # Orchestration only (~150 lines)
│   │   └── EventBus.js            # Event system
│   ├── services/
│   │   ├── LessonLoader.js        # JSON loading & caching
│   │   └── ProgressService.js     # Progress tracking
│   ├── components/
│   │   ├── QuizComponent.js       # Quiz UI & logic
│   │   ├── CodeEditor.js          # Code editing
│   │   ├── StepRenderer.js        # Step display
│   │   └── VisualComponent.js     # Visual examples
│   ├── utils/
│   │   ├── validators.js          # Code validation
│   │   ├── formatters.js          # Content formatting
│   │   └── schema-validator.js    # JSON schema validation
│   └── managers/
│       ├── StateManager.js        # Centralized state
│       └── ThemeManager.js        # Theme switching
├── lessons/                   # Lesson JSON files
│   ├── schemas/
│   │   └── lesson-schema.json     # NEW - Lesson data schema
│   ├── html/
│   ├── css/
│   └── js/
├── tests/                     # NEW - Test files
│   ├── unit/
│   │   ├── validators.test.js
│   │   └── formatters.test.js
│   └── e2e/
│       └── lessons.spec.js
└── .eslintrc.json            # NEW - Linting rules
```

**Phase 2: Build Tooling**
- [ ] Install Vite (`npm create vite@latest`)
- [ ] Set up hot reload
- [ ] Configure ES modules
- [ ] Add build scripts

**Phase 3: Testing**
- [ ] Install Vitest
- [ ] Write validation tests
- [ ] Write component tests
- [ ] Add E2E tests with Playwright

**Phase 4: Code Quality**
- [ ] Install ESLint + Prettier
- [ ] Fix all linting errors
- [ ] Add pre-commit hooks
- [ ] Document all public APIs

## Current File Structure

### JavaScript (Needs Refactoring)
```
js/
├── academy.js (403 lines)           # Main app initialization
├── tutorial-engine.js (1116 lines)  # ❌ TOO LARGE - needs splitting
├── progress-tracker.js (468 lines)  # OK size but could be modular
├── code-playground.js (378 lines)   # OK
├── challenge-editor.js (372 lines)  # OK
├── guided-editor.js (256 lines)     # OK
├── terminal-widget.js (235 lines)   # OK
├── visual-adjuster.js (235 lines)   # OK
└── theme-manager.js (124 lines)     # OK
```

### CSS (Acceptable but needs organization)
```
css/
├── themes.css        # CSS variables
├── academy.css       # Main styles
├── tier-components.css
├── terminal-widget.css
└── terminal-tasks.css
```

### Lessons (Good structure)
```
lessons/
├── html/
│   └── lesson-01.json   # 6 steps, properly structured
├── css/
├── js/
└── projects/
```

## Data Standards

### Lesson JSON Schema

ALL lesson JSON files MUST follow this schema:

```json
{
  "id": "html-01",
  "title": "Your First HTML Page",
  "description": "Learn the basics of HTML",
  "difficulty": "beginner",
  "estimatedTime": "10 minutes",
  "steps": [
    {
      "title": "Step Title",
      "content": "Markdown-like content with **bold**, `code`, and lists",
      "visual": "<div>HTML for visual examples</div>",
      "task": {
        "type": "quiz|code|completion|guided-code|challenge|terminal",
        "instruction": "What to do",
        "question": "Quiz question?",           // For quiz type
        "options": [...],                       // For quiz type
        "hint": "Helpful hint",
        "starter": "<h1></h1>",                 // For code type
        "solution": "<h1>My Page</h1>",         // For code type
        "validate": "(code) => code.includes('<h1>')"  // Validation function as string
      }
    }
  ]
}
```

**Validation Rule:** Before loading any lesson, validate it against the schema using `ajv` library.

### Visual Content Standards

Inline HTML in `visual` field MUST:
- Use inline styles (no external CSS dependencies)
- Use CSS variables for theming: `var(--primary-color)`
- Be self-contained and not affect page layout
- Font size ≤ 14px for content, ≤ 20px for headings
- Padding ≤ 15px

**Example:**
```html
"visual": "<div style=\"background:rgba(0,255,255,0.05);padding:12px;border-radius:6px;border:1px solid rgba(0,255,255,0.3);font-size:12px;\"><code>Example</code></div>"
```

## Code Validation System

### Current Implementation (tutorial-engine.js)
```javascript
// Validation functions stored as strings in JSON
"validate": "(code) => code.includes('<h1>') && code.includes('</h1>')"

// Executed with eval() - WORKS but not ideal
const validateFn = eval(task.validate);
const isValid = validateFn(code);
```

### Better Implementation (After Refactoring)
```javascript
// utils/validators.js
export const validators = {
    hasHTMLTag: (code, tag) => {
        const regex = new RegExp(`<${tag}>.*</${tag}>`, 's');
        return regex.test(code);
    },

    hasText: (code, text) => {
        return code.includes(text);
    },

    isValidHTML: (code) => {
        try {
            const parser = new DOMParser();
            const doc = parser.parseFromString(code, 'text/html');
            return !doc.querySelector('parsererror');
        } catch {
            return false;
        }
    }
};

// Lesson JSON references validator by name
"validate": {
    "type": "composite",
    "rules": [
        { "validator": "hasHTMLTag", "args": ["h1"] },
        { "validator": "hasText", "args": ["My First Page"] }
    ]
}
```

## Development Workflow

### Current (Simple HTTP Server)
```bash
cd /home/gh0st/dvn/code-academy
python3 -m http.server 8011
```

**Issues:**
- No hot reload
- Manual cache clearing required
- No build process
- No bundling/optimization

### Target (After Tooling Setup)
```bash
# Development
npm run dev        # Starts Vite dev server with hot reload

# Testing
npm test           # Run unit tests
npm run test:e2e   # Run end-to-end tests

# Production
npm run build      # Creates optimized dist/ folder
npm run preview    # Preview production build locally
```

## Known Issues

### Fixed (2026-01-10)
1. ✅ JSON files not loading - Was using embedded data in tutorial-engine.js
2. ✅ Step 1-3 content poor quality - Rewrote with proper explanations
3. ✅ Code editor too tall - Reduced from rows=8 to rows=4
4. ✅ Cache busting - Updated to ?v=5

### Outstanding
1. ❌ No modular architecture - Needs full refactor
2. ❌ No automated tests - Can't verify changes don't break existing features
3. ❌ No build tooling - Manual cache clearing required
4. ❌ No TypeScript - No type safety
5. ❌ No schema validation - Can't catch malformed lesson JSON

## Immediate Action Items

**BEFORE adding any new features:**

1. **Set up build tooling** (2 hours)
   ```bash
   npm init -y
   npm install --save-dev vite
   npm install --save-dev vitest @vitest/ui
   npm install --save-dev eslint prettier
   npm install ajv  # JSON schema validation
   ```

2. **Create modular structure** (4 hours)
   - Extract components from tutorial-engine.js
   - Create service layer for data loading
   - Implement centralized state management
   - Set up event bus for component communication

3. **Add testing** (2 hours)
   - Write validator tests
   - Write formatter tests
   - Add E2E test for lesson flow

4. **Documentation** (1 hour)
   - Write ARCHITECTURE.md
   - Document component APIs
   - Create contribution guide

**Total Time: ~9 hours to proper foundation**

## Mobile Considerations

When building features, consider:
- Touch targets ≥ 44px
- Font sizes ≥ 14px for readability
- Viewport-relative sizing
- No hover-only interactions
- Test on actual device (not just DevTools)

**Mobile Breakpoints:**
```css
/* Mobile */
@media (max-width: 767px) { }

/* Tablet */
@media (min-width: 768px) and (max-width: 1023px) { }

/* Desktop */
@media (min-width: 1024px) { }
```

## Performance Targets

- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Lesson load time: < 500ms
- Code validation: < 50ms
- Lighthouse score: > 90

## Browser Support

**Target:**
- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile Safari (iOS 14+)
- Chrome Mobile (latest)

**No support needed:**
- Internet Explorer
- Old Android browsers (< Android 8)

## Code Documentation Standards

Follow PKN standards from `/home/gh0st/pkn/CLAUDE.md`:

```javascript
// REQUIRED: Inline annotations
const lessonData = await this.loadLesson(id);  // Load from JSON file | ref:services/LessonLoader.js

// REQUIRED: JSDoc for all public methods
/**
 * Loads a lesson from JSON file and validates it
 * @param {string} id - Lesson ID (e.g., 'html-01')
 * @returns {Promise<Object>} Validated lesson data
 * @throws {Error} If lesson not found or validation fails
 */
async loadLesson(id) { ... }
```

## Git Workflow

### Branch Naming
```
feature/modular-architecture
feature/lesson-schema-validation
fix/code-editor-height
refactor/split-tutorial-engine
docs/architecture-guide
```

### Commit Messages
```
refactor(core): split tutorial-engine into modular components

- Created core/TutorialEngine.js (150 lines, orchestration only)
- Created services/LessonLoader.js (data loading + caching)
- Created components/QuizComponent.js (quiz UI)
- Created components/CodeEditor.js (code editing)
- Created utils/validators.js (validation functions)

BREAKING CHANGE: TutorialEngine is now a module export, not global

Reduces main file from 1116 lines to 150 lines
Enables unit testing of individual components
Follows architecture standards in /home/gh0st/dvn/ARCHITECTURE_STANDARDS.md
```

## Testing Strategy

### Unit Tests (Vitest)
Test individual utilities and validators:
```javascript
// tests/unit/validators.test.js
import { hasHTMLTag, hasText } from '../../src/utils/validators.js';

describe('hasHTMLTag', () => {
    it('detects valid h1 tag', () => {
        expect(hasHTMLTag('<h1>Title</h1>', 'h1')).toBe(true);
    });

    it('rejects incomplete tag', () => {
        expect(hasHTMLTag('<h1>Title', 'h1')).toBe(false);
    });
});
```

### Integration Tests
Test component interactions:
```javascript
// tests/integration/lesson-flow.test.js
test('completes a full lesson', async () => {
    const engine = new TutorialEngine();
    await engine.loadLesson('html-01');

    // Step 1: Quiz
    engine.selectAnswer(0);
    expect(engine.canProceed()).toBe(true);

    // Step 2: Code task
    engine.setCode('<h1>My Page</h1>');
    expect(engine.validateCurrentTask()).toBe(true);
});
```

### E2E Tests (Playwright)
Test full user journey:
```javascript
// tests/e2e/lesson.spec.js
test('complete HTML lesson 1', async ({ page }) => {
    await page.goto('http://localhost:8011');

    // Select path
    await page.click('text=HTML Fundamentals');

    // Start lesson
    await page.click('text=Your First HTML Page');

    // Step 1: Answer quiz
    await page.click('text=HyperText Markup Language');
    await page.click('text=Next');

    // Step 2: Write code
    await page.fill('.code-editor', '<h1>My First Page</h1>');
    await page.click('text=Check My Code');

    // Verify completion
    await expect(page.locator('.lesson-complete')).toBeVisible();
});
```

## Security Considerations

### Input Validation
- **NEVER** use `eval()` on user input
- Validate all lesson JSON against schema
- Sanitize HTML in visual examples
- Use textContent instead of innerHTML when possible

### Content Security Policy
```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               script-src 'self' 'unsafe-inline';
               style-src 'self' 'unsafe-inline';">
```

## Accessibility

- All interactive elements keyboard accessible
- ARIA labels on buttons
- Proper heading hierarchy
- Color contrast ratio ≥ 4.5:1
- Focus indicators visible

## Next Steps

**DO NOT ADD NEW FEATURES UNTIL:**
1. ✅ Read `/home/gh0st/dvn/ARCHITECTURE_STANDARDS.md`
2. ⏳ Set up proper build tooling (Vite)
3. ⏳ Refactor to modular architecture
4. ⏳ Add automated testing
5. ⏳ Document all components

**After proper foundation is in place, then:**
- Add more lessons (CSS, JavaScript, Projects)
- Implement advanced features (code hints, AI assistance)
- Add progress syncing across devices
- Build mobile app version

## Reference Projects

- **PKN** (`/home/gh0st/pkn/`) - Modular architecture example
- **PKN Mobile** (`~/pkn-phone/` on Termux) - Mobile optimization example
- **Architecture Standards** (`/home/gh0st/dvn/ARCHITECTURE_STANDARDS.md`) - Master guide

## Questions?

1. Check ARCHITECTURE_STANDARDS.md
2. Look at PKN reference implementation
3. Ask before making structural changes
4. When in doubt, prioritize maintainability over cleverness

**STRUCTURE BEFORE FEATURES - NO EXCEPTIONS**
