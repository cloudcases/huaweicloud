### BaseMatcher

```java
package org.hamcrest;

/**
 * BaseClass for all Matcher implementations.
 *
 * @see Matcher
 */
public abstract class BaseMatcher<T> implements Matcher<T> {

    /**
     * @see Matcher#_dont_implement_Matcher___instead_extend_BaseMatcher_()
     */
    @Override
    @Deprecated
    public final void _dont_implement_Matcher___instead_extend_BaseMatcher_() {
        // See Matcher interface for an explanation of this method.
    }

    @Override
    public void describeMismatch(Object item, Description description) {
        description.appendText("was ").appendValue(item);
    }

    @Override
    public String toString() {
        return StringDescription.toString(this);
    }

    /**
     * Useful null-check method. Writes a mismatch description if the actual object is null
     * @param actual the object to check
     * @param mismatch where to write the mismatch description, if any
     * @return false iff the actual object is null
     */
    protected static boolean isNotNull(Object actual, Description mismatch) {
        if (actual == null) {
            mismatch.appendText("was null");
            return false;
        }
        return true;
    }
}
```

### DiagnosingMatcher

```java
package org.hamcrest;

public abstract class DiagnosingMatcher<T> extends BaseMatcher<T> {

    @Override
    public final boolean matches(Object item) {
        return matches(item, Description.NONE);
    }

    @Override
    public final void describeMismatch(Object item, Description mismatchDescription) {
        matches(item, mismatchDescription);
    }

    protected abstract boolean matches(Object item, Description mismatchDescription);
}
```

### TypeSafeMatcher

```java
package org.hamcrest;

import org.hamcrest.internal.ReflectiveTypeFinder;

/**
 * Convenient base class for Matchers that require a non-null value of a specific type.
 * This simply implements the null check, checks the type and then casts.
 *
 * @author Joe Walnes
 * @author Steve Freeman
 * @author Nat Pryce
 */
public abstract class TypeSafeMatcher<T> extends BaseMatcher<T> {
    private static final ReflectiveTypeFinder TYPE_FINDER = new ReflectiveTypeFinder("matchesSafely", 1, 0);
    
    final private Class<?> expectedType;

    /**
     * The default constructor for simple sub types
     */
    protected TypeSafeMatcher() {
        this(TYPE_FINDER);
    }
   
    /**
     * Use this constructor if the subclass that implements <code>matchesSafely</code> 
     * is <em>not</em> the class that binds &lt;T&gt; to a type. 
     * @param expectedType The expectedType of the actual value.
     */
    protected TypeSafeMatcher(Class<?> expectedType) {
        this.expectedType = expectedType;
    }
    
    /**
     * Use this constructor if the subclass that implements <code>matchesSafely</code> 
     * is <em>not</em> the class that binds &lt;T&gt; to a type. 
     * @param typeFinder A type finder to extract the type
     */
    protected TypeSafeMatcher(ReflectiveTypeFinder typeFinder) {
      this.expectedType = typeFinder.findExpectedType(getClass()); 
    }
 
    /**
     * Subclasses should implement this. The item will already have been checked for
     * the specific type and will never be null.
     */
    protected abstract boolean matchesSafely(T item);
    
    /**
     * Subclasses should override this. The item will already have been checked for
     * the specific type and will never be null.
     */
    protected void describeMismatchSafely(T item, Description mismatchDescription) {
        super.describeMismatch(item, mismatchDescription);
    }
    
    /**
     * Methods made final to prevent accidental override.
     * If you need to override this, there's no point on extending TypeSafeMatcher.
     * Instead, extend the {@link BaseMatcher}.
     */
    @Override
    @SuppressWarnings({"unchecked"})
    public final boolean matches(Object item) {
        return item != null
                && expectedType.isInstance(item)
                && matchesSafely((T) item);
    }
    
    @SuppressWarnings("unchecked")
    @Override
    final public void describeMismatch(Object item, Description description) {
        if (item == null) {
            super.describeMismatch(null, description);
        } else if (! expectedType.isInstance(item)) {
            description.appendText("was a ")
                       .appendText(item.getClass().getName())
                       .appendText(" (")
                       .appendValue(item)
                       .appendText(")");
        } else {
            describeMismatchSafely((T)item, description);
        }
    }
}
```

### TypeSafeDiagnosingMatcher

```java
package org.hamcrest;

import org.hamcrest.internal.ReflectiveTypeFinder;


/**
 * Convenient base class for Matchers that require a non-null value of a specific type
 * and that will report why the received value has been rejected.
 * This implements the null check, checks the type and then casts. 
 * To use, implement <pre>matchesSafely()</pre>. 
 *
 * @param <T>
 * @author Neil Dunn
 * @author Nat Pryce
 * @author Steve Freeman
 */
public abstract class TypeSafeDiagnosingMatcher<T> extends BaseMatcher<T> {
    private static final ReflectiveTypeFinder TYPE_FINDER = new ReflectiveTypeFinder("matchesSafely", 2, 0); 
    private final Class<?> expectedType;

    /**
     * Subclasses should implement this. The item will already have been checked
     * for the specific type and will never be null.
     */
    protected abstract boolean matchesSafely(T item, Description mismatchDescription);

    /**
     * Use this constructor if the subclass that implements <code>matchesSafely</code> 
     * is <em>not</em> the class that binds &lt;T&gt; to a type. 
     * @param expectedType The expectedType of the actual value.
     */
    protected TypeSafeDiagnosingMatcher(Class<?> expectedType) {
      this.expectedType = expectedType;
    }

    /**
     * Use this constructor if the subclass that implements <code>matchesSafely</code> 
     * is <em>not</em> the class that binds &lt;T&gt; to a type. 
     * @param typeFinder A type finder to extract the type
     */
    protected TypeSafeDiagnosingMatcher(ReflectiveTypeFinder typeFinder) {
      this.expectedType = typeFinder.findExpectedType(getClass()); 
    }

    /**
     * The default constructor for simple sub types
     */
    protected TypeSafeDiagnosingMatcher() {
      this(TYPE_FINDER); 
    }

    @Override
    @SuppressWarnings("unchecked")
    public final boolean matches(Object item) {
        return item != null
            && expectedType.isInstance(item)
            && matchesSafely((T) item, new Description.NullDescription());
    }

    @SuppressWarnings("unchecked")
    @Override
    public final void describeMismatch(Object item, Description mismatchDescription) {
      if (item == null) {
        mismatchDescription.appendText("was null");
    } else if (!expectedType.isInstance(item)) {
      mismatchDescription.appendText("was ")
            .appendText(item.getClass().getSimpleName())
            .appendText(" ")
            .appendValue(item);
    } else {
        matchesSafely((T) item, mismatchDescription);
      }
    }
}
```

### FeatureMatcher

```java
package org.hamcrest;

import org.hamcrest.internal.ReflectiveTypeFinder;

/**
 * Supporting class for matching a feature of an object. Implement <code>featureValueOf()</code>
 * in a subclass to pull out the feature to be matched against. 
 *
 * @param <T> The type of the object to be matched
 * @param <U> The type of the feature to be matched
 */
public abstract class FeatureMatcher<T, U> extends TypeSafeDiagnosingMatcher<T> {
  private static final ReflectiveTypeFinder TYPE_FINDER = new ReflectiveTypeFinder("featureValueOf", 1, 0); 
  private final Matcher<? super U> subMatcher;
  private final String featureDescription;
  private final String featureName;
  
  /**
   * Constructor
   * @param subMatcher The matcher to apply to the feature
   * @param featureDescription Descriptive text to use in describeTo
   * @param featureName Identifying text for mismatch message
   */
  public FeatureMatcher(Matcher<? super U> subMatcher, String featureDescription, String featureName) {
    super(TYPE_FINDER);
    this.subMatcher = subMatcher;
    this.featureDescription = featureDescription;
    this.featureName = featureName;
  }
  
  /**
   * Implement this to extract the interesting feature.
   * @param actual the target object
   * @return the feature to be matched
   */
  protected abstract U featureValueOf(T actual);

  @Override
  protected boolean matchesSafely(T actual, Description mismatch) {
    final U featureValue = featureValueOf(actual);
    if (!subMatcher.matches(featureValue)) {
      mismatch.appendText(featureName).appendText(" ");
      subMatcher.describeMismatch(featureValue, mismatch);
      return false;
    }
    return true;
  }
      
  @Override
  public final void describeTo(Description description) {
    description.appendText(featureDescription).appendText(" ")
               .appendDescriptionOf(subMatcher);
  }
}
```

