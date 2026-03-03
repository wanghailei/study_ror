



## `cds-combo-box`

### `<cds-combo-box>` with Stimulus Key Learnings: 

1.  **Event Handling**:
    *   The custom event `cds-combo-box-input` can be unreliable for capturing typing in real-time depending on the version/implementation.
    *   The standard `input` event **does** bubble up from the Shadow DOM (due to event retargeting), making it the most reliable trigger for search logic.

2.  **Value Retrieval**:
    *   Getting the raw text value from a `<cds-combo-box>` while typing is tricky. `event.target.value` is often empty (as it tracks the *selected* item).
    *   The typed value is usually in the internal `<input>` element. Accessing it via `this.comboboxTarget.shadowRoot.querySelector('input').value` is a robust fallback.

3.  **Dynamic Items**:
    *   To update options asynchronously, we can't just pass a JSON array to a property easily in vanilla JS.
    *   The most reliable method is **DOM Manipulation**: clearing `innerHTML` and appending `<cds-combo-box-item>` elements. This forces the Web Component to re-index its slots.

4.  **Selection**:
    *   The `cds-combo-box-selected` event works well for capturing the final selection.
    *   The selected item is available in `event.detail.item`.





## cds-popover