title: Template strings, components, models and events
tags: [javascript, mozilla]
comments: https://github.com/jrburke/jrburke.com/issues/34
~
*This is a shorter, rougher post to outline a recent experiment around UI construction. It is a bit high level, and sets some context around the [htemplate](https://github.com/jrburke/htemplate) module*.

Some nice things about the [React](https://facebook.github.io/react/)-related world that I wanted to try for a work project:

1. A component system.
2. Grouping the DOM building with the JS code that knows about the state (In React, this is the render method, using `React.createElement` or [JSX](https://facebook.github.io/react/docs/displaying-data.html#jsx-syntax)).
3. From Flux, enforcing visual updates to be triggered from model changes, not from other visual components.

However, because of cultural constraints and an interest to keep the up front cost small, keep the stack small and shallow, I wanted to avoid a virtual DOM implementation and a transpiler.

What I ended up using:

1. [Custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements). This makes sense for the work project since we want to test out the custom element API, help find bugs. Custom elements are enabled by default for the project, so no special shims or adapters are necessary.

2. [htemplate](https://github.com/jrburke/htemplate): uses [tagged template strings](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/template_strings#Tagged_template_strings) to allow grouping the state logic in JS with building up the DOM. I get to use ES2015 without needing any build transforms to translate it to ES5, and I wanted to avoid the JSX transforms.

3. Adopt a cultural practice of calling the model API, then wait for the model object to update and bind the re-rendering of the custom element to events emitted from the model. The custom elements directly subscribe to the model to get changes instead of passing state down through components via a top level component.

In order to not take the cost of re-rending stuff that has not changed, the UI would be broken down into smaller custom elements that listened for fine-grained model updates.

This is possible because we have a front end object, called `model` which sits in front of the core backend model API, and if we notice that we need to group or constraint model update events to help scope visual updates, we have a place to do that.

A similar idea is behind [GraphQL](https://github.com/facebook/graphql) and [Falcor](https://github.com/Netflix/falcor), but this approach was done without a formal query language: construct a way to only see part of the whole model, scope data update events to subsets. Model properties/event names were used as the scoping mechanism.

## Custom element construction

This can be done in any number of ways, but I was already using [`element`](https://github.com/jrburke/element), so I just continued with it. However, I did not need to use the [`template`](https://github.com/jrburke/element#template-loader-plugin-custom-features) module, instead [`htemplate`](https://github.com/jrburke/htemplate) was used to construct the DOM within the components.

`element` supports building up the custom element prototype via mixins instead of inheritance, and if multiple mixins define methods for the [custom element lifecycle](https://github.com/jrburke/element#element-lifecycle-background), `element` will chain them together.

## Model construction

`model` is an object that mixes in an event emitter. There are lots of choices for event emitters. I used [`evt`](https://github.com/jrburke/evt) because it supports a `.latest()` concept, where it will call the listener if there is a current value for the event property, and for any future updates.

## Binding the model to the view

`element` mixins are used to bind the model updates to a `render()` method on the component. A sample of the end result:

* `base_render` sets up the relationship between htemplate and the model.
* `['accounts', 'folders']` are the model properties that this component wants to get updates about.
* The function is the "renderFn" given to `htemplate`.
* The html`` part is the tagged template string created by htemplate.
* The model object is passed to the custom element as `this.model`.

```javascript
{
  render: require('../base_render')(['accounts', 'folders'], function(html) {
    var currentAccount = this.model.account;
    if (!currentAccount) {
      return;
    }

    html`
    <a data-prop="accountHeader"
       data-dclick="toggleAccounts"
       class="fld-acct-header closed" role="region">
      <span class="fld-acct-header-account-label">${currentAccount.name}</span>
      <span class="fld-acct-header-account-header"
            data-l10n-id="drawer-accounts-header"></span>
      <span class="fld-account-switch-arrow"></span>
    </a>
    <div data-prop="fldAcctScrollInner" class="fld-acct-scrollinner">
      <div data-prop="fldAcctContainer" class="fld-acct-container">
        <!-- The list of accounts -->
        <div data-prop="accountContainer"
             data-dclick="onClickAccount"
             class="fld-accountlist-container collapsed">
        `;

        // Add DOM for each account.
        if (this.state.accounts) {
          this.state.accounts.items.forEach((account) => {
            // Highlight the account currently in use
            var selectedClass = this.model.account &&
                                this.model.account.id === account.id ?
                                'fld-account-selected' : '';

            html`
            <a class="fld-account-item ${selectedClass}"
               data-account-id="${account.id}">
              <span class="selected-indicator"></span>
              <span class="fld-account-name">${account.name}</span>
            </a>
            `;
          });
        }

        html`
        </div>
        <!-- The list of folders for the current account. -->
        <div data-prop="foldersContainer"
             data-dclick="onClickFolder"
             class="fld-folders-container">
        `;

          if (this.state.folders) {
            this.state.folders.items.forEach((folder) => {
              var extraClasses = [];

              if (!folder.selectable) {
                extraClasses.push('fld-folder-unselectable');
              }

              var depthIdx = Math.min(FOLDER_DEPTH_CLASSES.length - 1,
                                      folder.depth);
              extraClasses.push(FOLDER_DEPTH_CLASSES[depthIdx]);
              if (depthIdx > 0) {
                extraClasses.push('fld-folder-depthnonzero');
              }

              if (folder === this.model.folder) {
                extraClasses.push('fld-folder-selected');
              }

              html`
              <a class="fld-folder-item ${extraClasses.join(' ')}"
                 data-type="${folder.type}"
                 data-folder-id="${folder.id}">
                <span class="selected-indicator"></span>
                <span dir="auto"
                      class="fld-folder-name">${folder.name}</span>
                <span class="fld-folder-unread"></span>
              </a>
              `;
            });
          }

        html`
        </div>
      </div>
    </div>
    `;
  })
}
```

If the accounts or folders update frequently, then I could create smaller custom elements that focus on displaying those pieces based on the individual model property changes. However, for the time that this UI is shown, those model properties will rarely change, so inlining the work to show both lists in one component is fine.

## htemplate notes

htemplate supports passing non-string values to sub-elements by setting a property on the sub-element instead of setting a string attribute. This is useful to pass down objects, like the `model` object, to sub-elements. More details in the [Property setting section of the htemplate readme](#https://github.com/jrburke/htemplate#property-setting).

Editor support for syntax highlighting HTML inside the tagged template strings helps with the string legibility. It would be great to get more editors supporting it as I expect it will become  more common as tagged template strings get more visibility. Here are a couple of issues tracking that for different editors:

* [Atom editor](https://github.com/atom/language-javascript/pull/282)
* [Sublime Text, JavaScriptNext.tmLanguage package](https://github.com/Benvie/JavaScriptNext.tmLanguage/issues/134)

If you can, help your editor improve the display of a HTML string templates.

## Summary

I like the feel of it so far. As with most technology choices, it is about the tradeoffs you can accept.

I am fortunate to be able to use ES2015 and custom elements natively, not an option for many projects. It is fun to play in the future natively, I am excited to see those pieces become widely available across browsers.

Not using a virtual DOM implementation requires more thought on rate of updates for a component. Instead of just letting a React render pass sort out the details, The rate of model update events should be considered more, and possibly creating smaller components that care about finer grained model updates.

A virtual DOM can allow the developer to be more carefree about this, at the possible cost of React creating a larger set of internal objects to do a diff when the model changes only affect a small portion of the UI.

There are some cases where I do not want to just blast the innerHTML of the custom element on every model update. For instance, a CSS animated spinner that is activated by a class change to an element. In that case, I do not want to reset the innerHTML as the animating spinner would appear to jump around and reset. In those cases the custom element may decide to check if the existing DOM has the class set correctly instead of always resetting the innerHTML. A more manual diff model.

On the flip side, those cases are small and scoped, and the overall bootstrap code size of the project is smaller, with less build machinery in place.

So, tradeoffs. To be clear, the React ecosystem has a lot to offer, but it has been fun exploring an alternate approach inspired by some aspects of it but with different tradeoffs.
