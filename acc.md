# Accordion Integration Guide

## Architecture Overview

The `ui-accordion` is a **wrapper around Angular Material's MatExpansionPanel**, not a custom component from scratch. This gives us:

- **Robustness & A11y**: Keyboard nav, ARIA attributes, animations all handled by Material  
- **Stability**: Maintained by Google, proven in production  
- **Simple API**: Our wrapper exposes a clean input/output interface  
- **Isolation**: Features customize via inputs, never touch Material directly

### Component Structure

```
UiAccordionComponent (Container)
    ↓
    @ContentChildren detects children
    ↓
    Enforces mode: single/multiple
    ↓
    UiAccordionItemComponent (Item wrapper layer)
        ↓
        MatExpansionPanel (Material – animations, state, a11y)
```

---

## UiAccordionItemComponent: All Inputs & Outputs

Here are **all the inputs** you can pass to each `<ui-accordion-item>`:

| Input | Type | Purpose | Example |
|-------|------|---------|---------|
| `header` | string (optional) | Simple text header | `header="Documents"` |
| `headerTemplate` | TemplateRef | Custom header via ng-template | `[headerTemplate]="myHeader"` |
| `headerContext` | any | Data passed to headerTemplate | `[headerContext]="{ icon, title }"` |
| `isSelected` | boolean | Adds `.ui-accordion-item__header--selected` CSS class | `[isSelected]="true"` |
| `disabled` | boolean | Greys out, prevents interaction | `[disabled]="false"` |
| `opened` | boolean | Item starts expanded | `[opened]="true"` |
| `ariaLabel` | string | Accessibility label for screen readers | `[ariaLabel]="'Documents panel'"` |

| Output | Emits |
|--------|-------|
| `openedChange` | `'expanded'` or `'collapsed'` when user toggles |

---

## Integration: Step by Step

### 1. Import from public index

```ts
import { UiAccordionComponent, UiAccordionItemComponent } 
  from 'app/shared/ui/ui-accordion';

@Component({
  imports: [
    CommonModule,
    UiAccordionComponent,
    UiAccordionItemComponent,
  ]
})
export class MyContainer { }
```

**Always** use the index, never `.../accordion/` or `.../accordion-item/` directly.

### 2. Container manages selection state

Selection logic lives in your **container**, not in the accordion:

```ts
export class MailboxContainer {
  // Signal controls which item is selected
  selection = signal<{ section: boolean; mailboxId: number | null }>({ 
    section: true,  // Section "Mes courriers" is selected by default
    mailboxId: null // No mailbox selected
  });

  selectSection() {
    this.selection.set({ section: true, mailboxId: null });
    this.router.navigate(['/mailboxes']);
  }

  selectMailbox(id: number) {
    this.selection.set({ section: false, mailboxId: id });
    this.router.navigate(['/mailboxes', id]);
  }
}
```

The container is responsible for:
- Managing the selection state
- Binding `isSelected` to each item
- Responding to clicks `(openedChange)`

### 3. Basic template: simple header + content

```html
<ui-accordion [mode]="'single'" [variant]="'bordered'">
  
  <!-- Item with simple string header -->
  <ui-accordion-item 
    header="My Simple Header"
    [isSelected]="selection().section"
    (openedChange)="selectSection()">
    <p>Content projections here</p>
  </ui-accordion-item>

</ui-accordion>
```

---

## How to customize: Header Template + Context

### Simple approach: `header` input

```html
<ui-accordion-item header="Documents">
  <!-- content -->
</ui-accordion-item>
```

### Advanced: Custom header with `headerTemplate` + `headerContext`

**In your TypeScript:**

```ts
export class MyContainer {
  faEnvelope = faEnvelopeOpen; // FontAwesome icon
  
  headerData = signal({ 
    icon: this.faEnvelope, 
    title: 'Mes courriers',
    unreadCount: 5
  });
}
```

**In your template:**

```html
<ui-accordion [mode]="'single'">
  
  <ui-accordion-item 
    [headerTemplate]="customHeader"
    [headerContext]="headerData()"
    [isSelected]="selection().section"
    (openedChange)="selectSection()">
    <p>Section content here</p>
  </ui-accordion-item>

</ui-accordion>

<!-- Define the custom header template -->
<ng-template #customHeader let-data>
  <!-- let-data = receives headerContext() as $implicit -->
  <fa-icon [icon]="data.icon" class="mr-2"></fa-icon>
  <span class="font-bold">{{ data.title }}</span>
  <span class="badge ml-auto">{{ data.unreadCount }}</span>
</ng-template>
```

**Key**: `let-data` in ng-template receives `headerContext()` as the implicit variable.

---

## Content Projection (inside accordion-item)

Everything between `<ui-accordion-item>` tags is automatically projected inside the item's body:

```html
<ui-accordion-item header="Users">
  
  <!-- All of this is projected via <ng-content> -->
  <ul>
    <li>Alice</li>
    <li>Bob</li>
  </ul>
  <button (click)="addUser()">Add User</button>

</ui-accordion-item>
```

You can project:
- HTML elements
- Components
- Structural directives (`*ngIf`, `*ngFor`)
- Ng-templates

---

## Styling the selected state

When `isSelected="true"`, Material adds the CSS class `.ui-accordion-item__header--selected` to the header.

**Override it with `::ng-deep`:**

```scss
// in your container.scss
::ng-deep .ui-accordion .mat-expansion-panel-header.ui-accordion-item__header--selected {
  background-color: $color-selection-bg-list !important;
  border-left: 4px solid $color-primary !important;
  font-weight: 600 !important;
}
```

**Why `::ng-deep`**: Material's styles are view-encapsulated; you must pierce the boundary.  
**Why `!important`**: Material's specificity is high; force your styles to win.

---

## Real-world Example: Mailbox with Custom Headers

**TypeScript:**

```ts
@Component({
  selector: 'app-mailbox-container',
  templateUrl: './mailbox.container.html',
  styleUrl: './mailbox.container.scss',
  imports: [UiAccordionComponent, UiAccordionItemComponent, CommonModule]
})
export class MailboxContainer {
  faEnvelopeOpen = faEnvelopeOpen;

  selection = signal<{ section: boolean; mailboxId: number | null }>({ 
    section: true, 
    mailboxId: null 
  });

  mailboxes = toSignal(
    this.mailboxService.mailboxes$.pipe(
      map(sections => sections.map(s => ({
        ...s,
        mailboxes: s.mailboxes.map(m => ({
          ...m,
          initials: this.avatarUtils.getInitials(m.label)
        }))
      })))
    ),
    { initialValue: [] }
  );

  selectSection() {
    this.selection.set({ section: true, mailboxId: null });
    this.router.navigate(['/mailboxes']);
  }

  selectMailbox(id: number) {
    this.selection.set({ section: false, mailboxId: id });
    this.router.navigate(['/mailboxes', id]);
  }
}
```

**Template:**

```html
<ui-accordion [mode]="'single'" [variant]="'bordered'">

  <!-- "Mes courriers" section with icon header -->
  <ui-accordion-item 
    [headerTemplate]="sectionHeader"
    [headerContext]="{ icon: faEnvelopeOpen, title: 'Mes courriers' }"
    [isSelected]="selection().section"
    (openedChange)="selectSection()">
    <p>All mailboxes section</p>
  </ui-accordion-item>

  <!-- Individual mailboxes -->
  @for (mailbox of mailboxes()[0]?.mailboxes || []; track mailbox.id) {
    <ui-accordion-item 
      [headerTemplate]="mailboxHeader"
      [headerContext]="{ mailbox }"
      [isSelected]="selection().mailboxId === mailbox.id"
      (openedChange)="selectMailbox(mailbox.id)">
      <p>{{ mailbox.description }}</p>
    </ui-accordion-item>
  }

</ui-accordion>

<!-- Header templates -->
<ng-template #sectionHeader let-data>
  <fa-icon [icon]="data.icon" class="mr-3"></fa-icon>
  <span>{{ data.title }}</span>
</ng-template>

<ng-template #mailboxHeader let-data>
  <div class="w-8 h-8 mr-3 rounded flex items-center justify-center bg-gray-200">
    {{ data.mailbox.initials }}
  </div>
  <span>{{ data.mailbox.label }}</span>
</ng-template>
```

**Styles (in mailbox.container.scss):**

```scss
::ng-deep .ui-accordion .mat-expansion-panel-header.ui-accordion-item__header--selected {
  background-color: $color-selection-bg-list !important;
  border-left: 4px solid $color-primary !important;
}
```

---

## Common Integration Mistakes

1. **Importing from nested paths** instead of index  
   ❌ `import { UiAccordionComponent } from '.../accordion/accordion.component'`  
   ✅ `import { UiAccordionComponent } from 'app/shared/ui/ui-accordion'`

2. **Trying to manage selection inside the item**  
   ❌ Item tries to call `this.isSelected.set(true)`  
   ✅ Container passes `[isSelected]="selection().something"`

3. **Forgetting `::ng-deep` for header styling**  
   Result: Your styles don't apply because Material's encapsulation wins

4. **Confusing `header` and `headerTemplate`**  
   ❌ Using both at once  
   ✅ Use `header="text"` for simple text, OR `[headerTemplate]` for custom layouts

5. **Not understanding content projection**  
   Forgetting that everything inside `<ui-accordion-item>` is automatically projected

---

## Checklist before shipping

- [ ] Import from `app/shared/ui/ui-accordion` (not deep paths)
- [ ] Add components to `imports` array
- [ ] Selection state managed in container (not item)
- [ ] Custom headers use `ng-template` + `let-data`
- [ ] Styling uses `::ng-deep` + `!important` for Material headers
- [ ] Content projected correctly inside `<ui-accordion-item>` tags
- [ ] Using `[isSelected]` binding to control header styles

---

See `features/mailboxes/containers/mailbox/mailbox.container.ts` for the full working example.
