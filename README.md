# React Testing Best Practices

## Big Picture
The obvious goal of writing tests is to prevent future bugs. A less obvious goal is the freedom to make changes quickly because of the confidence you can have in your changes. This makes having good test coverage extremely valuable when working in a CI/CD environment.

## What tests add value?
In many programming applications the tests that need to be written are obvious, we have logic with inputs and outputs to assert on. When working with frontend applications it's often not quite as simple-- we simply have more to test before we can have the same level of confidence. Within our frontends we should be testing the HTML output, User Interactions, and any API contract we have with another party (including our backends).

## Line Coverage vs Branch Coverage
Branch coverage is a better indicator of how well the code has been tested. It reports whether or not the logical branches are covered which is more often than not the thing we should be testing. Although I'd argue branch coverage is more important, looking at both provides the actual picture of what's going on.

Considering the code below, if you only wrote a test with `thing` being true you would get nearly 50% line coverage but only 20% branch coverage.  However, if you wrote the tests for everything other than `thing` you'd end up with 80% branch while maintaining the 50% of lines. If you only considered the branch coverage you _might_ think this is good enough, but by keeping both scores in perspective you can see that more testing needs to be done.

```
if (thing) {
	//do
	//lots
	//of
	//stuff
	//here
} else if (thing2) {
	//do one thing
} else if (thing3) {
	//do one thing
} else if (thing4) {
	//do one thing
} else {
	//do one thing
}
```

## Strategies

### Pure Unit Tests (Jest)
*Useful for: Utilities*

There will always be functions that we'll use to help us format a string or get data from a complex data structure or something similar. We should try to keep those functions in their own files to make testing them easier. Having these functions in their own files will also improve discoverability later in case we want to do the same operation elsewhere.

### DOM Snapshot Tests (Jest + React Testing Library)
*Useful for: HTML, User Interactions, API Contracts*

For **HTML** it is useful to visualize the complete changes and see the git diff between outputs. Not only is it faster to write a snapshot test for testing what is on the screen, it will catch all changes (including changes in components you depend on) and asserts on all things the user is seeing rather than making a developer remember to look for each block of text. As noted, this will catch changes from the components that make be used to compose the one you are testing which will help understand what all is being affected by the change of a component. I *highly* recommend having at least one standard assertion before the snapshot assertion to hedge against updating all snapshots and overlooking an obvious bug.

Like HTML test, **User Interactions** can be tested incredibly quickly and effectively by using snapshot tests. I like to write tests with the following pattern...

```
describe("SomeComponent", () => {
	describe("Submit Button onClick", () => {
		// setup and teardown mocks before/after
		
		it ("displays success message to the user after successful submission", () => {
			// tell mock to return success
			
			const { container } = render(<SomeComponent />)
			
			expect(container).toMatchSnapshot('initial render')
			
			fireEvent(getByText(container, 'Submit'), new MouseEvent('click', { bubbles: true }))
			
			expect(fooMock).toHaveBeenCalledTimes(1)
			expect(fooMock.mock.calls[0][0]).toMatchSnapshot('fooMock args')
			expect(container).toMatchSnapshot('after submit success')
		})
	})
})
```
These kinds of tests will ensure that the correct functions are being applied to the correct event listeners as well as partially testing what those functions are doing (you will often want to mock them).


For **API Contracts**, it is useful to mock out the service and assert on the data being passed to them. I find snapshot tests especially useful for this because it will catch all of the changes without needing to remember to add them. Consider an API that has a test written asserting we send the following data...

```
{
	type: "foo",
	data: "bar"
}
```

... but we later add an additional property `meta: "verse"`. If we had written...
```
expect(request.type).toBe("foo")
expect(request.data).toBe("bar")
```
...not only is that more verbose than simply making a snapshot but it also wouldn't catch our new property and protect the code from future changes.

### Integration Tests (Selenium, Cypress)
*Useful for: Verifying Common Flows*

Once an app is stable and we have clear expectations it becomes useful to codify those expectations in a series of Integration Tests. This reduces the amount of time our QA engineers need to spend constantly regression testing the same user flows. These tests should be defined by our QA engineers and should represent the flows that they are commonly testing. *These tests should not be written lightly*, they are expensive to maintain because they incorporate all pieces of the puzzle. At my previous employer we used Selenium to drive very specific tests around very specific user flows (ie confirm the user can hit the landing page, navigate to a product detail page, add the item to their cart, etc). These kinds of test catch all sorts of useful bugs like javascript somehow not compiling but they are fairly brittle.

### User Experience Tests (Lighthouse)

Lighthouse is an incredibly useful tool for web devs to make sure that what we're producing is a delight for our users. It will aid in discovering problems around Performance, PWA, Best Practices, Accessibility and SEO. Most sites on the web actually get fairly poor scores on Lighthouse which is one way we can have an edge over our competitors.

It is important to note that not all pages are created equal. Each site will have different pages that need to see more priority related to performance optimization. It is much more important for the landing page to be fast than any other page, in most cases. I would love to strive for performance scores of 85+ on all pages and well into the 90s for SEO. Lighthouse gives great feedback for proposed changes to improve your scores-- right now most of advice for thechosen.tv is around image delivery.

I recommend always running Lighthouse tests in the Mobile device mode. This causes them to do some throttling and test in a more real-world scenario. If our sites are performing well for mobile devices they should be an incredibly delightful experience on a desktop.
