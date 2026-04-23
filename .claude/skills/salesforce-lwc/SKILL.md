---
name: salesforce-lwc
description: LWC development skills for Salesforce — component scaffolding, wire adapters, parent-child communication, imperative Apex calls, Jest testing, security rules, and accessibility standards.
---

# Salesforce LWC Skill

Trigger phrases: "create a lwc component", "new lightning web component", "add a wire adapter", "create an lwc", "lwc jest test", "lwc component for", "secure this lwc", "wire this to apex"

---

## Skill: scaffold-lwc-component

### Pre-creation checklist
- Component name must be camelCase (e.g. `orderSummary`, `invoiceList`)
- Target contexts: App Page, Record Page, Home Page, Utility Bar, Flow Screen — affects `targets` in meta-xml
- Identify if it needs: wire adapters, imperative Apex calls, parent-child communication, navigation

### Folder structure
```
force-app/main/default/lwc/<componentName>/
  <componentName>.html
  <componentName>.js
  <componentName>.js-meta.xml
  <componentName>.css
  __tests__/
    <componentName>.test.js
```

### HTML template
```html
<template>
    <!-- component markup here -->
</template>
```
- Never use `lwc:dom="manual"` unless absolutely required
- Never set `innerHTML` — use `lwc:if`, `lwc:for`, and template directives
- Use SLDS utility classes for spacing, typography, and layout

### JavaScript class
```javascript
import { LightningElement, api, wire, track } from 'lwc';

export default class ComponentName extends LightningElement {
    // @api exposes a property to parent components or App Builder
    @api recordId;

    // @wire auto-refreshes when tracked properties change
    // @track is only needed for nested object/array mutation detection
}
```

### Meta-xml template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>66.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__RecordPage</target>
        <target>lightning__AppPage</target>
        <target>lightning__HomePage</target>
    </targets>
</LightningComponentBundle>
```

### Post-creation
```
sf project deploy start --metadata LightningComponentBundle:<componentName> --target-org $SF_TARGET_ORG
```

---

## Skill: wire-adapters

### Wire to a standard Salesforce adapter
```javascript
import { LightningElement, wire } from 'lwc';
import { getRecord, getFieldValue } from 'lightning/uiRecordApi';
import NAME_FIELD from '@salesforce/schema/Account.Name';

export default class MyComponent extends LightningElement {
    @api recordId;

    @wire(getRecord, { recordId: '$recordId', fields: [NAME_FIELD] })
    wiredRecord;

    get accountName() {
        return getFieldValue(this.wiredRecord.data, NAME_FIELD);
    }
}
```

### Wire to an Apex method
```javascript
import { LightningElement, wire } from 'lwc';
import getItems from '@salesforce/apex/MyController.getItems';

export default class MyComponent extends LightningElement {
    @wire(getItems)
    wiredItems;

    get items() {
        return this.wiredItems.data ?? [];
    }

    get error() {
        return this.wiredItems.error;
    }
}
```
- The Apex method must be `@AuraEnabled(cacheable=true)` for `@wire` use
- Always handle both `.data` and `.error` — never assume success
- Use `??` (nullish coalescing) instead of `||` to safely default empty data

### Imperative Apex call (when you need a user action to trigger the call)
```javascript
import getItems from '@salesforce/apex/MyController.getItems';

async handleClick() {
    try {
        const result = await getItems({ param: this.value });
        this.items = result;
    } catch (error) {
        this.error = error.body?.message ?? 'An unexpected error occurred.';
    }
}
```
- Always wrap in `try/catch`
- Never expose raw `error.message` to users — use `error.body?.message`
- Use `async/await` over `.then()/.catch()` chains for readability

---

## Skill: parent-child-communication

### Parent → Child (property binding)
```html
<!-- parent.html -->
<c-child-component record-id={recordId}></c-child-component>
```
```javascript
// childComponent.js
import { LightningElement, api } from 'lwc';
export default class ChildComponent extends LightningElement {
    @api recordId;
}
```
- `@api` properties are read-only from inside the child — never mutate them directly
- HTML attribute names are kebab-case; JS property names are camelCase — the framework maps between them

### Parent → Child (method call)
```javascript
// parent.js
this.template.querySelector('c-child-component').doSomething();

// childComponent.js
@api doSomething() {
    // public method callable by parent
}
```

### Child → Parent (custom event)
```javascript
// childComponent.js
this.dispatchEvent(new CustomEvent('itemselected', { detail: { id: this.itemId } }));

// parent.html
<c-child-component onitemselected={handleItemSelected}></c-child-component>

// parent.js
handleItemSelected(event) {
    const selectedId = event.detail.id;
}
```
- Event names must be lowercase (no camelCase, no hyphens in the name itself)
- Use `detail` to pass data — keep it serializable (no class instances)
- Use `bubbles: true, composed: true` only when the event needs to cross shadow DOM boundaries

---

## Skill: lwc-jest-testing

### Setup
```javascript
// __tests__/<componentName>.test.js
import { createElement } from 'lwc';
import ComponentName from 'c/componentName';

describe('c-component-name', () => {
    afterEach(() => {
        while (document.body.firstChild) {
            document.body.removeChild(document.body.firstChild);
        }
    });

    it('renders correctly', () => {
        const element = createElement('c-component-name', { is: ComponentName });
        document.body.appendChild(element);
        expect(element.shadowRoot.querySelector('h1')).not.toBeNull();
    });
});
```

### Mock Apex
```javascript
import getItems from '@salesforce/apex/MyController.getItems';
jest.mock('@salesforce/apex/MyController.getItems', () => ({ default: jest.fn() }), { virtual: true });

it('displays items on success', async () => {
    getItems.mockResolvedValue([{ Id: '001', Name: 'Test' }]);
    const element = createElement('c-my-component', { is: MyComponent });
    document.body.appendChild(element);
    await Promise.resolve();
    // assert rendered items
});
```

### Mock navigation
```javascript
import { CurrentPageReference } from 'lightning/navigation';
jest.mock('lightning/navigation', () => require('@salesforce/slds-jest/src/lightning-stubs/navigation/navigation'), { virtual: true });
```

### Required test scenarios
1. **Happy path** — data loads and renders correctly
2. **Error state** — Apex returns error, component shows error message
3. **Empty state** — empty list/null data handled gracefully
4. **User interaction** — button click, event dispatch, method call

Run Jest tests:
```
npm run test:unit
```

---

## Skill: lwc-security-and-accessibility

### Security rules
- Never use `eval()` or `new Function()`
- Never set `element.innerHTML` — use template directives (`lwc:if`, `lwc:for`)
- Never use `document.querySelector` — use `this.template.querySelector` within shadow DOM
- Never hardcode record IDs, org IDs, or URLs — use `@salesforce/label` or pass via `@api`
- Never store sensitive data in `localStorage` or `sessionStorage`

### Accessibility rules
- Every interactive element must have an accessible label: use `aria-label`, `aria-labelledby`, or visible `<label>`
- Use `<lightning-*>` base components where possible — they are WCAG 2.1 AA compliant by default
- Ensure keyboard navigation works: `Tab`, `Enter`, `Escape` for modals and dropdowns
- Never rely on color alone to convey meaning — pair with icons or text
- Test with a screen reader or use `axe` in Jest: `expect(element).toBeAccessible()`

### SLDS standards
- Use SLDS utility classes for spacing (`slds-m-*`, `slds-p-*`) — no inline `style` attributes
- Use SLDS design tokens for colors and typography — no hardcoded hex values or `px` font sizes
- Use `<lightning-card>`, `<lightning-button>`, `<lightning-input>` — avoid reimplementing base component behavior
