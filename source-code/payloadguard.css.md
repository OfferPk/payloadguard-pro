/\* PayloadGuard Pro \- Modern Dark UI Stylesheet \*/

:root {  
  \--bg-primary: \#0b0f19;  
  \--bg-secondary: \#111827;  
  \--bg-tertiary: \#1f2937;  
  \--bg-card: \#161e2e;  
  \--bg-input: \#0d131f;

  \--border-color: \#2d3748;  
  \--border-focus: \#3b82f6;

  \--text-primary: \#f3f4f6;  
  \--text-secondary: \#9ca3af;  
  \--text-muted: \#6b7280;

  \--color-brand: \#3b82f6;  
  \--color-success: \#10b981;  
  \--color-danger: \#ef4444;  
  \--color-warning: \#f59e0b;  
  \--color-zod: \#3b82f6;  
  \--color-pydantic: \#10b981;  
  \--color-ts: \#3178c6;  
  \--color-go: \#00add8;

  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;

  \--radius-sm: 6px;  
  \--radius-md: 10px;  
  \--radius-lg: 16px;  
  \--shadow-md: 0 4px 12px rgba(0, 0, 0, 0.5);  
  \--shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.6);  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-primary);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  min-height: 100vh;  
  display: flex;  
  flex-direction: column;  
}

/\* Header \*/  
header.app-header {  
  background-color: var(--bg-secondary);  
  border-bottom: 1px solid var(--border-color);  
  padding: 0.85rem 1.75rem;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  position: sticky;  
  top: 0;  
  z-index: 50;  
}

.brand-wrapper {  
  display: flex;  
  align-items: center;  
  gap: 0.75rem;  
}

.brand-icon {  
  width: 32px;  
  height: 32px;  
  background: linear-gradient(135deg, \#3b82f6, \#10b981);  
  border-radius: var(--radius-sm);  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-weight: 700;  
  font-size: 1.1rem;  
  color: white;  
}

.brand-title {  
  font-size: 1.25rem;  
  font-weight: 700;  
  letter-spacing: \-0.025em;  
  background: linear-gradient(90deg, \#60a5fa, \#34d399);  
  \-webkit-background-clip: text;  
  \-webkit-text-fill-color: transparent;  
}

.brand-tag {  
  font-size: 0.7rem;  
  background-color: rgba(59, 130, 246, 0.15);  
  color: \#60a5fa;  
  padding: 0.15rem 0.5rem;  
  border-radius: 9999px;  
  border: 1px solid rgba(59, 130, 246, 0.3);  
  font-weight: 600;  
  text-transform: uppercase;  
}

.header-actions {  
  display: flex;  
  align-items: center;  
  gap: 1rem;  
}

.form-select {  
  background-color: var(--bg-input);  
  color: var(--text-primary);  
  border: 1px solid var(--border-color);  
  padding: 0.45rem 0.85rem;  
  border-radius: var(--radius-sm);  
  font-size: 0.85rem;  
  outline: none;  
  cursor: pointer;  
}

.btn {  
  display: inline-flex;  
  align-items: center;  
  gap: 0.4rem;  
  padding: 0.45rem 0.95rem;  
  border-radius: var(--radius-sm);  
  font-size: 0.85rem;  
  font-weight: 600;  
  cursor: pointer;  
  border: 1px solid transparent;  
  transition: all 0.2s ease;  
}

.btn-primary {  
  background: linear-gradient(135deg, \#2563eb, \#3b82f6);  
  color: white;  
}  
.btn-primary:hover {  
  background: linear-gradient(135deg, \#1d4ed8, \#2563eb);  
}

.btn-secondary {  
  background-color: var(--bg-tertiary);  
  color: var(--text-primary);  
  border-color: var(--border-color);  
}  
.btn-secondary:hover {  
  background-color: \#374151;  
}

/\* Main Grid Layout \*/  
main.app-main {  
  flex: 1;  
  display: grid;  
  grid-template-columns: 1fr 1.25fr;  
  gap: 1.5rem;  
  padding: 1.5rem;  
  max-width: 1700px;  
  margin: 0 auto;  
  width: 100%;  
}

@media (max-width: 1100px) {  
  main.app-main {  
    grid-template-columns: 1fr;  
  }  
}

/\* Studio Card \*/  
.studio-card {  
  background-color: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-md);  
  box-shadow: var(--shadow-md);  
  display: flex;  
  flex-direction: column;  
  overflow: hidden;  
}

.card-header {  
  padding: 0.85rem 1.25rem;  
  background-color: rgba(31, 41, 55, 0.5);  
  border-bottom: 1px solid var(--border-color);  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.card-title {  
  font-size: 0.95rem;  
  font-weight: 700;  
  display: flex;  
  align-items: center;  
  gap: 0.5rem;  
}

.card-body {  
  padding: 1.25rem;  
  display: flex;  
  flex-direction: column;  
  gap: 1.25rem;  
}

/\* Status Banner \*/  
.status-banner {  
  padding: 0.75rem 1rem;  
  border-radius: var(--radius-sm);  
  font-size: 0.85rem;  
  font-weight: 600;  
  display: flex;  
  align-items: center;  
  justify-content: space-between;  
}

.banner-valid {  
  background-color: rgba(16, 185, 129, 0.15);  
  color: \#34d399;  
  border: 1px solid rgba(16, 185, 129, 0.3);  
}

.banner-invalid {  
  background-color: rgba(239, 68, 68, 0.15);  
  color: \#f87171;  
  border: 1px solid rgba(239, 68, 68, 0.3);  
}

/\* Split Textareas \*/  
.editor-grid {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 1rem;  
}

@media (max-width: 768px) {  
  .editor-grid {  
    grid-template-columns: 1fr;  
  }  
}

.editor-box {  
  display: flex;  
  flex-direction: column;  
  gap: 0.4rem;  
}

.editor-label {  
  font-size: 0.8rem;  
  font-weight: 600;  
  color: var(--text-secondary);  
}

textarea.json-textarea {  
  background-color: var(--bg-input);  
  color: var(--text-primary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 0.85rem;  
  font-family: var(--font-mono);  
  font-size: 0.8rem;  
  min-height: 260px;  
  resize: vertical;  
  line-height: 1.45;  
  outline: none;  
}

textarea.json-textarea:focus {  
  border-color: var(--border-focus);  
}

/\* Errors list \*/  
.error-chip {  
  background-color: rgba(239, 68, 68, 0.15);  
  border: 1px solid rgba(239, 68, 68, 0.3);  
  color: \#f87171;  
  padding: 0.4rem 0.75rem;  
  border-radius: var(--radius-sm);  
  font-family: var(--font-mono);  
  font-size: 0.78rem;  
  display: flex;  
  gap: 0.5rem;  
}

/\* Tabs \*/  
.tabs-nav {  
  display: flex;  
  border-bottom: 1px solid var(--border-color);  
  background-color: rgba(17, 24, 39, 0.7);  
}

.tab-btn {  
  padding: 0.75rem 1.25rem;  
  background: none;  
  border: none;  
  color: var(--text-secondary);  
  font-size: 0.85rem;  
  font-weight: 600;  
  cursor: pointer;  
  border-bottom: 2px solid transparent;  
  transition: all 0.2s;  
  display: flex;  
  align-items: center;  
  gap: 0.4rem;  
}

.tab-btn:hover {  
  color: var(--text-primary);  
}

.tab-btn.active {  
  color: \#60a5fa;  
  border-bottom-color: \#3b82f6;  
  background-color: rgba(59, 130, 246, 0.05);  
}

.tab-content {  
  display: none;  
  padding: 1.25rem;  
  flex-direction: column;  
  gap: 1.25rem;  
}

.tab-content.active {  
  display: flex;  
}

/\* Sub-tabs for Frameworks \*/  
.subtabs-bar {  
  display: flex;  
  gap: 0.5rem;  
}

.subtab-btn {  
  padding: 0.35rem 0.75rem;  
  border-radius: var(--radius-sm);  
  font-size: 0.8rem;  
  font-weight: 600;  
  cursor: pointer;  
  background-color: var(--bg-tertiary);  
  border: 1px solid var(--border-color);  
  color: var(--text-secondary);  
}

.subtab-btn.active {  
  background-color: var(--color-brand);  
  color: white;  
  border-color: \#3b82f6;  
}

/\* Fuzz Matrix List \*/  
.fuzz-list {  
  display: flex;  
  flex-direction: column;  
  gap: 0.75rem;  
  max-height: 480px;  
  overflow-y: auto;  
}

.fuzz-card {  
  background-color: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-left: 4px solid var(--color-warning);  
  border-radius: var(--radius-sm);  
  padding: 0.85rem;  
  display: flex;  
  flex-direction: column;  
  gap: 0.35rem;  
}

.fuzz-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.fuzz-title {  
  font-size: 0.85rem;  
  font-weight: 700;  
  color: var(--text-primary);  
}

.severity-pill {  
  font-size: 0.65rem;  
  font-weight: 700;  
  padding: 0.15rem 0.45rem;  
  border-radius: 4px;  
  text-transform: uppercase;  
}

.pill-REJECT { background-color: rgba(239, 68, 68, 0.2); color: \#f87171; }  
.pill-CHECK { background-color: rgba(245, 158, 11, 0.2); color: \#fbbf24; }  
.pill-SEC { background-color: rgba(139, 92, 246, 0.2); color: \#c084fc; }

pre.code-preview {  
  background-color: var(--bg-input);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 1rem;  
  font-family: var(--font-mono);  
  font-size: 0.8rem;  
  line-height: 1.5;  
  overflow-x: auto;  
  max-height: 420px;  
}

/\* Footer \*/  
footer.app-footer {  
  background-color: var(--bg-secondary);  
  border-top: 1px solid var(--border-color);  
  padding: 1rem 1.75rem;  
  font-size: 0.8rem;  
  color: var(--text-secondary);  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  margin-top: auto;  
}

@media print {  
  header.app-header, .tabs-nav, .header-actions, button {  
    display: none \!important;  
  }  
  main.app-main {  
    display: block;  
    padding: 0;  
  }  
  .studio-card {  
    border: none;  
    box-shadow: none;  
  }  
  .tab-content {  
    display: block \!important;  
  }  
}  
