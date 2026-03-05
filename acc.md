# Accordion Integration Guide

## Architecture Overview

The `ui-accordion` is a **wrapper around Angular Material's MatExpansionPanel**, not a custom component from scratch. This hybrid approach gives us:

- **Robustness**: A11y, keyboard nav, animations all handled by Material  
- **Stability**: Maintained by Google, battle-tested across projects  
- **Flexibility**: Our wrapper layer (inputs like `mode`, `variant`) keeps the interface simple while Material handles complexity underneath  
- **Isolation**: Every feature that wants to use accordion customizes it through our wrapper, never directly touching Material

### Component Structure

```
UiAccordionComponent (Container wrapper)
    ↓
    @ContentChildren → detects all UiAccordionItemComponent children
    ↓
    Enforces "single" vs "multiple" mode
    ↓
    UiAccordionItemComponent (Item wrapper)
        ↓
        MatExpansionPanel (Material – handles animations, a11y, state)
```

Each `UiAccordionItemComponent` sits on top of `MatExpansionPanel`, adding a thin layer for theming and selection tracking.

---

## Integration Steps

### 1. Importing from the public index

Always import from the public index so consumers don’t have to know internal paths:

```ts
import { UiAccordionComponent, UiAccordionItemComponent } from 'app/shared/ui/ui-accordion';
```

Place these components in the `imports` array of any standalone component or NgModule that will use the accordion.

**Why**: The index is the public API contract. If internals move (accordion/ vs accordion-item/), only the index changes, not your code.

### 2. Basic HTML structure

```html
<ui-accordion [mode]="'single'" [variant]="'bordered'">
  <ui-accordion-item header="Title 1" [isSelected]="selectedId() === 1" (openedChange)="onOpen(1)">
    Content 1
  </ui-accordion-item>

  <ui-accordion-item header="Title 2" [isSelected]="selectedId() === 2" (openedChange)="onOpen(2)">
    Content 2
  </ui-accordion-item>
</ui-accordion>
```

- `mode` can be `'single'` (only one item open) or `'multiple'`.
- `variant` controls the visual style; most projects use `'default'` or `'bordered'`.

## 3. Styling notes

Material’s encapsulated styles need to be overridden with a selector that penetrates the component. Example to change the header background:

```scss
/* in your feature stylesheet */
::ng-deep .ui-accordion .mat-expansion-panel-header.my-selected {
  background-color: $color-selection-bg-list !important;
}
```

`::ng-deep` (or `!important`) is required otherwise Angular Material wins. Do **not** remove this directive.

## 4. Selection logic

If you want to mark an item as "selected" based on route or other state, expose a signal (`selectedId()` or similar) at the container level and bind it to the `isSelected` input.

The shared component does **not** manage selection automatically; it only adds `ui-accordion-item__header--selected` class when the boolean is true.

Example of route‑based selection (see mailbox container for full recipe):

```ts
private extractMailboxId = (url: string) => { ... };
private currentUrl = toSignal(...Router.events...);
selection = computed(() => {
  const id = this.extractMailboxId(this.currentUrl());
  return { section: id===null, mailboxId: id };
});
```

## 5. Common pitfalls

1. **Skipping index exports** – importing from deep paths will break when internals move.
2. **Forgetting `imports` entries** – results in unknown element errors.
3. **Ignoring `::ng-deep`** – custom styles won’t apply.
4. **Attempting to mutate `isSelected` inside the item** – state must come from parent/route.
5. **Using the component without OnPush** – it’s designed for OnPush and you may lose performance.

## 6. Further reading

- See `ui-accordion/DOCUMENTATION.md` for full implementation details.
- Look at `features/mailboxes/containers/mailbox/mailbox.container.ts` for a real-world example of selection and routing.

---

Keep this guide handy when adding the accordion to new features; following these steps ensures correct behavior and avoids the bugs we tracked down during DSG‑1930.
