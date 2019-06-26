# Chapter 15 - Junit Internals

Firstly this chapter is not about TDD or JUnit. It’s about refactoring JUnit framework originally written by Erich Gamma and Kent Beck. Uncle Bob says after three hours of high-altitude work, they had written the basics of JUnit.

Uncle Bob selects ComparisonCompactor class as an example and he tries to refactor it. I think the idea behind this chapter is encouraging people to do refactoring. As you may guess before starting refactor, he says that ComparisonCompactor class has 100% test coverage which is vital for refactoring process.


## Case Introduction

We will look into bit of code that helps identify string comparision errors. This module is called ComparisionCompactor. It will shows us few refactoring rules which we can use in our day to day coding.

## Two files which we are going to use  1. ComparisionCompactor.java and 2. ComparisionCompactorTest.java
Although the unit test code has covered everything from the unit test perspective but we have scope to refine or refacor original ComparisionCompactor class

## The code before refactoring
```java
package junit.framework;

public class ComparisonCompactor {
	private static final String ELLIPSIS = "...";
	private static final String DELTA_END = "]";
	private static final String DELTA_START = "[";
	private int fContextLength;
	private String fExpected;
	private String fActual;
	private int fPrefix;
	private int fSuffix;

	public ComparisonCompactor(int contextLength, String expected, String actual) {
		fContextLength = contextLength;
		fExpected = expected;
		fActual = actual;
	}

	public String compact(String message) {
		if (fExpected == null || fActual == null || areStringsEqual())
			return Assert.format(message, fExpected, fActual);
		findCommonPrefix();
		findCommonSuffix();
		String expected = compactString(fExpected);
		String actual = compactString(fActual);
		return Assert.format(message, expected, actual);
	}

	private String compactString(String source) {
		String result = DELTA_START + source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
		if (fPrefix > 0)
			result = computeCommonPrefix() + result;
		if (fSuffix > 0)
			result = result + computeCommonSuffix();
		return result;
	}

	private void findCommonPrefix() {
		fPrefix = 0;
		int end = Math.min(fExpected.length(), fActual.length());
		for (; fPrefix < end; fPrefix++) {
			if (fExpected.charAt(fPrefix) != fActual.charAt(fPrefix))
				break;
		}
	}

	private void findCommonSuffix() {
		int expectedSuffix = fExpected.length() - 1;
		int actualSuffix = fActual.length() - 1;
		for (; actualSuffix >= fPrefix && expectedSuffix >= fPrefix; actualSuffix--, expectedSuffix--) {
			if (fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix))
				break;
		}
		fSuffix = fExpected.length() - expectedSuffix;
	}

	private String computeCommonPrefix() {
		return (fPrefix > fContextLength ? ELLIPSIS : "")
				+ fExpected.substring(Math.max(0, fPrefix - fContextLength), fPrefix);
	}

	private String computeCommonSuffix() {
		int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength, fExpected.length());
		return fExpected.substring(fExpected.length() - fSuffix + 1, end)
				+ (fExpected.length() - fSuffix + 1 < fExpected.length() - fContextLength ? ELLIPSIS : "");
	}

	private boolean areStringsEqual() {
		return fExpected.equals(fActual);
	}
}
```
## Refactoring Rules

**1. Remove unwanted prefix & suffix from variable names (N6: Avoid Encodings)**

_Names should not be encoded with type or scope information. Prefixes such as m or f are useless in today’s environments._

**2. Encapsulate condition wherever required (G28: Encapsulate Conditionals)**

_Boolean logic is hard enough to understand without having to see it in the context of an if or while statement. Extract functions that explain the intent of the conditional_

**3. Function name should show the intent (N7: Names Should Describe Side-Effects)**

_Names should describe everything that a function, variable, or class is or does. Don’t hide side effects with a name. Don’t use a simple verb to describe a function that does more than just that simple action._

**4. Variable names should be unambiguous (N4: Unambiguous Names)**

_Choose names that make the workings of a function or variable unambiguous_

**5. Function name should not contain negative words or intent (G29: Avoid Negative Conditionals)**

_Negatives are just a bit harder to understand than positives. So, when possible, conditionals should be expressed as positives._

**6. Function should be compact enough to its intended work (G30: Functions Should Do One Thing)**

_It is often tempting to create functions that have multiple sections that perform a series of operations. Functions of this kind do more than one thing, and should be converted into many smaller functions, each of which does one thing._

**7. Writing consistant conventions (G11: Inconsistency)**

_If you do something a certain way, do all similar things in the same way._ 

**8. N1: Choose Descriptive Names**

_Don’t be too quick to choose a name. Make sure the name is descriptive. Remember that meanings tend to drift as software evolves, so frequently reevaluate the appropriateness of the names you choose._

**9. Expose temporal coupling and use tight coupling (G31: Hidden Temporal Couplings)**

_Temporal couplings are often necessary, but you should not hide the coupling. Structure the arguments of your functions such that the order in which they should be called is obvious._

**10. Name of variable should be unique (G32: Don’t Be Arbitrary)**

_Have a reason for the way you structure your code, and make sure that reason is communicated by the structure of the code. If a structure appears  arbitrary, others will feel empowered to change it._

**11. G33: Encapsulate Boundary Conditions**

_Boundary conditions are hard to keep track of. Put the processing for them in one place._

**12. G9: Dead Code**

_Dead code is code that isn’t executed._

## The code after refactoring
```java
package junit.framework;

public class ComparisonCompactor {
	private static final String ELLIPSIS = "...";
	private static final String DELTA_END = "]";
	private static final String DELTA_START = "[";
	private int contextLength;
	private String expected;
	private String actual;
	private int prefixLength;
	private int suffixLength;

	public ComparisonCompactor(int contextLength, String expected, String actual) {
		this.contextLength = contextLength;
		this.expected = expected;
		this.actual = actual;
	}

	public String formatCompactedComparison(String message) {
		String compactExpected = expected;
		String compactActual = actual;
		if (shouldBeCompacted()) {
			findCommonPrefixAndSuffix();
			compactExpected = compact(expected);
			compactActual = compact(actual);
		}
		return Assert.format(message, compactExpected, compactActual);
	}

	private boolean shouldBeCompacted() {
		return !shouldNotBeCompacted();
	}

	private boolean shouldNotBeCompacted() {
		return expected == null || actual == null || expected.equals(actual);
	}

	private void findCommonPrefixAndSuffix() {
		findCommonPrefix();
		suffixLength = 0;
		for (; !suffixOverlapsPrefix(); suffixLength++) {
			if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength))
				break;
		}
	}

	private char charFromEnd(String s, int i) {
		return s.charAt(s.length() - i - 1);
	}

	private boolean suffixOverlapsPrefix() {
		return actual.length() - suffixLength <= prefixLength || expected.length() - suffixLength <= prefixLength;
	}

	private void findCommonPrefix() {
		prefixLength = 0;
		int end = Math.min(expected.length(), actual.length());
		for (; prefixLength < end; prefixLength++)
			if (expected.charAt(prefixLength) != actual.charAt(prefixLength))
				break;
	}

	private String compact(String s) {
		return new StringBuilder().append(startingEllipsis()).append(startingContext()).append(DELTA_START)
				.append(delta(s)).append(DELTA_END).append(endingContext()).append(endingEllipsis()).toString();
	}

	private String startingEllipsis() {
		return prefixLength > contextLength ? ELLIPSIS : "";
	}

	private String startingContext() {
		int contextStart = Math.max(0, prefixLength - contextLength);
		int contextEnd = prefixLength;
		return expected.substring(contextStart, contextEnd);
	}

	private String delta(String s) {
		int deltaStart = prefixLength;
		int deltaEnd = s.length() - suffixLength;
		return s.substring(deltaStart, deltaEnd);
	}

	private String endingContext() {
		int contextStart = expected.length() - suffixLength;
		int contextEnd = Math.min(contextStart + contextLength, expected.length());
		return expected.substring(contextStart, contextEnd);
	}

	private String endingEllipsis() {
		return (suffixLength > contextLength ? ELLIPSIS : "");
	}
}
```

## Conclusion
- According to Boy Scout Rule, we have to leave the module bit cleaner, as no module is immune from improvement.

- Refactoring is an iterative process full of trial and error

- The idea behind this chapter is encouraging people to do refactoring once  %100 test coverage is reached, which is vital for refactoring process

- Keep your code as clean and simple as it can be. Never let the rot get started.


