### MatcherAssert

```java
package org.hamcrest;

public class MatcherAssert {
    public static <T> void assertThat(T actual, Matcher<? super T> matcher) {
        assertThat("", actual, matcher);
    }
    
    public static <T> void assertThat(String reason, T actual, Matcher<? super T> matcher) {
        if (!matcher.matches(actual)) {
            Description description = new StringDescription();
            description.appendText(reason)
                       .appendText(System.lineSeparator())
                       .appendText("Expected: ")
                       .appendDescriptionOf(matcher)
                       .appendText(System.lineSeparator())
                       .appendText("     but: ");
            matcher.describeMismatch(actual, description);
            
            throw new AssertionError(description.toString());
        }
    }
    
    public static void assertThat(String reason, boolean assertion) {
        if (!assertion) {
            throw new AssertionError(reason);
        }
    }
}
```