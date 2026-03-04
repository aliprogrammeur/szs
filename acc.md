# Accordion Integration Guide

This short guide explains how to properly include the shared `ui-accordion` component in a new feature. Follow the steps below and refer to the full `DOCUMENTATION.md` for deeper understanding.

## 1. Importing

Always import from the public index so consumers don’t have to know internal paths:

```ts
import { UiAccordionComponent, UiAccordionItemComponent } from 'app/shared/ui/ui-accordion';
```

Place these components in the `imports` array of any standalone component or NgModule that will use the accordion.

## 2. Basic usage

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
