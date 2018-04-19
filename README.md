# ProseMirror Cookbook
A series of short examples for understanding ProseMirror.

## Editor
#### Get notified of updates and changes
A plugin to know when something has changed in the editor (cursor position, selection, etc) and hook up on the updates:
```js
new Plugin({
  view() {
    return {
      update (update) {
        // do something here such as eventBus.dispatch('NEW-UPDATE', update)
      }
    }
  }
})
```
You can also hook up on the transactions:
```js
const editorElement = document.querySelector('#my-editor');
const contentElement = document.querySelector('#my-content');

const editorView = new EditorView(editorElement, {
  state: EditorState.create({
    doc: DOMParser.fromSchema(schema).parse(contentElement),
    plugins: plugins // an array of plugins
  }),
  dispatchTransaction (transaction) {
    const newState = editorView.state.apply(transaction);
    editorView.updateState(newState);
    // do something here such as eventBus.dispatch('NEW-TRANSACTIION', transaction)
  }
});
```

## Commands
#### Check the current active marks
A helper function that returns an array with the active marks in the current selection. This is helpful if you have a custom menu and want to highlight the buttons of the marks that are applied (bold, italic, etc):
```js
getCurrentMarks (editorView) {
  const state = editorView.state;
  const from = state.selection.from;
  const to = state.selection.to;

  // Set instead of array because there can't be duplicated values
  const selectionMarks = new Set();

  state.doc.nodesBetween(from, to, (node) => {
    node.marks.forEach((mark) => {
      selectionMarks.add(mark.type.name);
    });
  });

  return Array.from(selectionMarks);
}
```
#### Check the current available node types
A helper function that returns an array with the node types that can be applied in the current selection or cursor position. This is helpful for example if you have a custom menu and want to enable/disable/highlight buttons to switch from heading to paragraph.
```js
getAvailableBlockTypes (editorView, schema) {
  // get all the available nodeTypes in the schema
  const nodeTypes = schema.nodes;

  // iterate all the nodeTypes and check which ones can be applied
  return Object.keys(nodeTypes).filter((key) => {
    const nodeType = nodeTypes[key];
    // setBlockType() returns a function which returns false when a node can't be applied
    return setBlockType(nodeType)(this.editorView.state, null, this.editorView);
  });
}
```

## Decorations
#### Apply a decoration where the cursor is
A plugin that applies a decoration to the node where the cursor is:
```js
new Plugin({
  props: {
    decorations(state) {
      const selection = state.selection;
      const resolved = state.doc.resolve(selection.from);
      const decoration = Decoration.node(resolved.before(), resolved.after(), {class: 'selected'});
      // equivalent to
      // const decoration = Decoration.node(resolved.start() - 1, resolved.end() + 1, {class: 'selected'});
      return DecorationSet.create(state.doc, [decoration]);
    }
  }
})
```
#### Apply a decoration to the selected node(s)
A plugin for applying a decoration to the all the nodes that are "touched" by a selection:
```js
new Plugin({
  props: {
    decorations(state) {
      const selection = state.selection;
      const decorations = [];

      state.doc.nodesBetween(selection.from, selection.to, (node, position) => {
        if (node.isBlock) {
          decorations.push(Decoration.node(position, position + node.nodeSize, {class: 'selected'}));
        }
      });

      return DecorationSet.create(state.doc, decorations);
    }
  }
})
```
