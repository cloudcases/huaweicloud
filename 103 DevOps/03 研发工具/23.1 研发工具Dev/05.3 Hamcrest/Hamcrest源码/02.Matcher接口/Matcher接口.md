### Matcher

```java
package org.hamcrest;

public interface Matcher<T> extends SelfDescribing {

    /**
     * Evaluates the matcher for argument <var>item</var>.
     *
     * This method matches against Object, instead of the generic type T. This is
     * because the caller of the Matcher does not know at runtime what the type is
     * (because of type erasure with Java generics). It is down to the implementations
     * to check the correct type.
     *
     * @param actual the object against which the matcher is evaluated.
     * @return <code>true</code> if <var>item</var> matches, otherwise <code>false</code>.
     *
     * @see BaseMatcher
     */
    boolean matches(Object actual);
    
    void describeMismatch(Object actual, Description mismatchDescription);

    @Deprecated
    void _dont_implement_Matcher___instead_extend_BaseMatcher_();
}
```