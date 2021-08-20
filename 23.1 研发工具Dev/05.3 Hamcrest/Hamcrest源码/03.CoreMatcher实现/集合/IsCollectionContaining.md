### IsCollectionContaining

```java
package org.hamcrest.core;

import org.hamcrest.Description;
import org.hamcrest.Matcher;
import org.hamcrest.TypeSafeDiagnosingMatcher;

/**
 * @deprecated As of release 2.1, replaced by {@link IsIterableContaining}.
 */
@Deprecated
public class IsCollectionContaining<T> extends TypeSafeDiagnosingMatcher<Iterable<? super T>> {
    private final IsIterableContaining<T> delegate;

    public IsCollectionContaining(Matcher<? super T> elementMatcher) {
        this.delegate = new IsIterableContaining<>(elementMatcher);
    }

    @Override
    protected boolean matchesSafely(Iterable<? super T> collection, Description mismatchDescription) {
        return delegate.matchesSafely(collection, mismatchDescription);
    }

    @Override
    public void describeTo(Description description) {
        delegate.describeTo(description);
    }

    public static <T> Matcher<Iterable<? super T>> hasItem(Matcher<? super T> itemMatcher) {
        return IsIterableContaining.hasItem(itemMatcher);
    }

    public static <T> Matcher<Iterable<? super T>> hasItem(T item) {
        // Doesn't forward to hasItem() method so compiler can sort out generics.
        return IsIterableContaining.hasItem(item);
    }

    @SafeVarargs
    public static <T> Matcher<Iterable<T>> hasItems(Matcher<? super T>... itemMatchers) {
        return IsIterableContaining.hasItems(itemMatchers);
    }
    
    @SafeVarargs
    public static <T> Matcher<Iterable<T>> hasItems(T... items) {
        return IsIterableContaining.hasItems(items);
    }
}
```
