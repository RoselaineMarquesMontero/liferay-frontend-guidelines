# Recommendations for testing

## Testing UI components

### On shapshots

> It's nice to have a few general snapshots of how a component renders, but ... the more snapshots you have, the less helpful they are.
>
> — [@bryceosterhaus](https://github.com/liferay/liferay-frontend-guidelines/issues/49#issuecomment-506774669)

Snapshots SHOULD only be used as "smoke tests": they are very good for verifying that a component renders at all, but they are _not_ good for communicating intent or what is important. For example, when something in a large snapshot changes, there is nothing intrinsic in the snapshot to communicate whether any particular part of it is important.

This doesn't mean that snapshots are bad — a snapshot test is better than no tests at all — but that they should be used sparingly, and complemented with more-targeted assertions.

### On `data-testid` attributes

-   Shipping test attributes to the client is overtly bad because it increases network payloads, parse times, DOM bloat etc, and ends up becoming an unofficial API.
    -   Therefore, we MUST NOT send "data-testid" to the client.
-   The code and markup that is exercised in the test environment should match that which is used in the production environment, otherwise you are not actually testing the thing that you would like to test.
    -   Because of the recommendation above, any usage of "data-testid" MUST be stripped from when outside the test environment.
    -   But because of the desire to not have divergent code paths, "data-testid" SHOULD NOT be used, except as an interim measure.

> A "data-testid" says "this thing is important to me". But if it is important, why isn't it targetable in some user-visible way? That is, users care about behavior and content of specific things (buttons, sliders, charts etc). If the only way I can label something as important is via _a mechanism that is invisible to the user_ (not even in the source code on the client), then there is a risk that I might be testing an implementation detail and not something which a user actually cares about.

## Targeting elements without `data-testid`

-   In the absence of "data-testid", we seek the most stable way to target elements in tests ("stable" in the sense that tests are less likely to break as a result of superficial changes); we think the methods that are most likely to remain stable are the following and therefore recommend that we SHOULD use:
    -   Attributes or content that are significant to the user (for example, the text of a button such as "Publish"); at the end of the day, the visible (or audible) content and behavior of a product is literally the reason we build products, so the highest value tests should focus on aspects that are perceptible and observable to the user. In the button example, if the text changes, then we _want_ the test to break.
    -   `id` attributes when a semantically meaningful and unique "identity" can be assigned to an element (obvious considerations about multiple instances in a document needing to be unique apply).
    -   `class` based selectors where the name is semantically meaningful (ie. describing the role of the element as opposed to controlling the presentational aspects of it).
    -   other selectors, with a strong preference for _semantic_ elements because these are less likely to change over time (that is a list `<ul>` is likely to remain a list, and a `<nav>` element is likely to continue being for navigation, totally unlike arbitrary container elements like `<span>` or `<div>` that could change at any time).

### Writing user-centric tests

In [liferay-portal](https://github.com/liferay/liferay-portal) we bundle [@testing-library/user-event](https://github.com/testing-library/user-event/) via [liferay-npm-scripts](https://github.com/liferay/liferay-npm-scripts) so that we can write UI tests that express user actions in a high-level, expressive fashion in terms of clicking, typing and so on (instead of a low-level `fireEvent()` API). Not only does this train our focus on what matters (user and product behavior, instead of implementation details), as a bonus it shields us from the nitty-gritty details of how (in what combinations and in what order) browsers fire events.

## Start `it()` descriptions with a verb, not with "should"

Descriptions that start with "should" can usually be rewritten more concisely and without loss of information by dropping the "should" and starting with an appropriate verb. This is useful because it reduces the likelihood we'll have to introduce an ugly linebreak that harms readability:

## Example

Write this:

```javascript
it('applies default settings if none are given', () => {
	// ...
});
```

instead of:

```javascript
it('should apply default settings if none are given', () => {
	// ...
});
```

## Enforcement

[eslint-config-liferay](https://github.com/liferay/eslint-config-liferay) provide a custom lint rule, [liferay/no-it-should](https://github.com/liferay/eslint-config-liferay/blob/master/plugins/eslint-plugin-liferay/docs/rules/no-it-should.md), to guard against the use of "should" at the start of `it()` descriptions. It is active by default in all projects that use eslint-config-liferay.