---
Stage: Accepted
Start Date:
Release Date: Unreleased
Release Versions:
  ember-source: vX.Y.Z
  ember-data: vX.Y.Z
Relevant Team(s):
RFC PR:
---

<!---
Directions for above:

Stage: Leave as is
Start Date: Fill in with today's date, YYYY-MM-DD
Release Date: Leave as is
Release Versions: Leave as is
Relevant Team(s): Fill this in with the [team(s)](README.md#relevant-teams) to which this RFC applies
RFC PR: Fill this in with the URL for the Proposal RFC PR
-->

# <RFC title>

## Summary

Augment the named blocks syntax to allow conditional passing of named blocks.
This will help easily build layouts where different styles and markup are applied based on passed in blocks into a component.

## Motivation

Named blocks have been a great addition to the glimmer-vm and have enabled us to create a wide range of components with lots of customizable sections.
The syntax is missing a very key feature that has made using it feel cumbersome at times.

Consider a Card component that has an optional header section that applies some styling to the header by default,

```hbs
<section class="card">
  {{#if (has-block "header")}}
    <header style="height: 30px; background: green;">
      {{yield to="header"}}
    </header>
  {{/if}}
  <div class="card__body">
    {{yield to="body"}}
  </div>
</section>
```

If a consumer of this component wants to optionally include the header yield block and not include the header styles, currently they end up having to write their component as follows,

```hbs
{{#if this.showHeader}}
  <Card>
    <:header>My Card Header</:header>
    <:body>Card Details</:body>
  </Card>
{{else}}
  <Card>
    <:body>Card Details</:body>
  </Card>
{{/if}}
```

This form of inclusion results in many bugs over time as developers forget to update both sides of the conditional. Also as we add more optional blocks the number of conditionals and their possible permutations increase dramatically which make the component even harder to read and maintain.


Another alternative that is often used is modifying the Card component arguments API to accept a `showHeader` argument instead of using the has-block helper.

```hbs
<section class="card">
  {{#if @showHeader}}
    <header style="height: 30px; background: green;">
      {{yield to="header"}}
    </header>
  {{/if}}
  <div class="card__body">
    {{yield to="body"}}
  </div>
</section>
```

The consuming components code now looks as follows
```hbs
<Card @showHeader={{this.showHeader}}>
  <:header>My Card Header</:header>
  <:body>Card Details</:body>
</Card>
```

This form of usage is also unclear to the reader. As it looks like the header named block is yielded into the component however its actual inclusion is gated on the `showHeader` argument which is being passed to the component.

In contrast to the above current usages with conditional named blocks we would have very clean and easy to read component code for both the components.

## Detailed design

Allow conditionals in the named blocks invocations to compile and pass down any blocks that are evaluated to the child component.

```hbs
<Card>
  {{#if this.showHeader}}
    <:header>My Card Header</:header>
    <:meta>Card Meta<:meta>
  {{/if}}
  <:body>Card Details</:body>
</Card>
```

The `"{{has-block}}` helper should also continue to work as expected i.e. it should only evaluate to true if the conditional caused the block to be invoked.

## How we teach this

This can be a follow up after teaching named blocks to showcase how named blocks can be used to create flexible layouts and components very easily. The syntax itself is intuitive as its similar to any other conditional that exist in the templates.

## Drawbacks

This could complicate the implementation of named blocks in the vm and add additional maintenance over ahead for the core team going forward as it is another feature that the vm has to support.

## Alternatives

Move the conditional out of the component invocation. This will work but as brought up in the motivation section will lead to a lot of duplicated code that will be hard to maintain over time for component authors.

Pass in an argument to the component as a replacement to the `{{has-block}}` check. This will lead to components to be not as readable as to a reader it will seem like the blocks are always passed in but their inclusion is gated on a different argument passed into the component.
