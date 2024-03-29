## Every

```java
package org.hamcrest.core;

import org.hamcrest.Description;
import org.hamcrest.Matcher;
import org.hamcrest.TypeSafeDiagnosingMatcher;

public class Every<T> extends TypeSafeDiagnosingMatcher<Iterable<? extends T>> {
    private final Matcher<? super T> matcher;

    public Every(Matcher<? super T> matcher) {
        this.matcher= matcher;
    }

    @Override
    public boolean matchesSafely(Iterable<? extends T> collection, Description mismatchDescription) {
        for (T t : collection) {
            if (!matcher.matches(t)) {
                mismatchDescription.appendText("an item ");
                matcher.describeMismatch(t, mismatchDescription);
                return false;
            }
        }
        return true;
    }

    @Override
    public void describeTo(Description description) {
        description.appendText("every item is ").appendDescriptionOf(matcher);
    }

    /**
     * Creates a matcher for {@link Iterable}s that only matches when a single pass over the
     * examined {@link Iterable} yields items that are all matched by the specified
     * <code>itemMatcher</code>.
     * For example:
     * <pre>assertThat(Arrays.asList("bar", "baz"), everyItem(startsWith("ba")))</pre>
     * 
     * @param itemMatcher
     *     the matcher to apply to every item provided by the examined {@link Iterable}
     */
    public static <U> Matcher<Iterable<? extends U>> everyItem(final Matcher<U> itemMatcher) {
        return new Every<>(itemMatcher);
    }
}
```

