---
title: Configuration
redirect_from:
  - /lenskit-groovy/index.html
---

# Configuring a LensKit Recommender

[LenskitRecommenderEngineFactory]: http://lenskit.grouplens.org/apidocs/org/grouplens/lenskit/core/LenskitRecommenderEngineFactory.html

One of LensKit's goals is to be highly configurable with regards to
the algorithms used, choice of parameters for them, and various
algorithmic decisions for each algorithm (e.g. the similarity function
used for k-NN collaborative filtering, or the normalization applied to
ratings). [LenskitRecommenderEngineFactory][] is the main entry point
for configuring a recommender.

Recommender configuration is done by selecting the correct
implementation for various ''components'' (typically defined by Java
interfaces), and values for ''parameters''.  Pretty much every object
you can interact with in a LensKit recommender is a component, and
many of them use other components behind the scenes to do their
work. In the example code in GettingStarted, we find this line:

~~~java
factory.bind(RatingPredictor.class).to(ItemItemRatingPredictor.class);
~~~

[ItemItemRatingPredictor]: http://lenskit.grouplens.org/apidocs/org/grouplens/knn/item/ItemItemRatingPredictor.html
[RatingPredictor]: http://lenskit.grouplens.org/apidocs/org/grouplens/lenskit/RatingPredictor.html
[Guice]: https://code.google.com/p/google-guice/
[WP:DI]: http://en.wikipedia.org/wiki/Dependency_injection

If you are familiar with [dependency injection][WP:DI], particularly
with [Guice][], this line will look familiar.  What it does is tell
LensKit that we want to use [ItemItemRatingPredictor][] as the
implementation of the [RatingPredictor][] component. When our code
then asks the recommender for a rating predictor, it will use an
`ItemItemRatingPredictor`. Likewise, any other components that use a
`RatingPredictor`, such as an `ItemItemRecommender`, will use the
`ItemItemRatingPredictor`.

[ItemItemRecommender]: http://lenskit.grouplens.org/apidocs/org/grouplens/lenskit/knn/item/ItemItemRecommender.html
[Grapht]: http://github.com/grouplens/grapht

When you look at the JavaDoc for a component implementation, such as
[ItemItemRecommender][], you will see that it takes the components it
uses (its ''dependencies'') as parameters to its constructor or,
occasionally, parameters to setter methods. This is because LensKit is
built using the Dependency Injection design pattern. The
`LenskitRecommenderEngineFactory` provides ''automatic dependency
injection'', built using the [Grapht][] dependency injection
container. It automatically instantiates the various components in
accordance with the configuration (bindings) you provide in order to
create the recommender you desire.  Most components and parameters
have default settings, so LensKit will “just work” if you specify the
predictor and/or recommender you want to use, but you can always swap
out components for ones more suited to your application as necessary.

### Contexts

One feature provided by Grapht, and used heavily by LensKit, is
''context-sensitive'' bindings. These are bindings that choose how to
configure a component based on where that component is being
used. Formerly, these types of configurations were expressed with role
annotations; we are moving to heavier use of contexts because they
provide a cleaner, more easily discoverable solution in most cases.

To bind in a context, use the `within` method:

~~~java
factory.within(SimpleNeighborhoodFinder.class)
       .bind(UserVectorNormalizer.class)
       .to(BaselineSubtractingNormalizer.class);
~~~

[SimpleNeighborhoodFinder]: http://lenskit.grouplens.org/apidocs/org/grouplens/lenskit/knn/user/SimpleNeighborhoodFinder.html

This uses the baseline-subtracting normalizer as the vector
normalizer, but only when building the [SimpleNeighborhoodFinder][] or
one of its dependencies. It does not configure the normalizer passed
to the rating predictor — if no other bindings are present, then the
that is kept at the default.

Context-sensitive bindings override other bindings, so if you have a
non-contextual binding of `VectorNormalizer`, that binding still
applies everywhere except where the context is in active — that is,
everywhere except in `SimpleNeighborhoodFinder` or one of its
dependencies. You can have multiple context-based bindings, and you
can also chain contexts in bindings. The closest, longest matching
chain of contexts determines the actual binding to use.

Grapht also provides an `at` method, in addition to `within`; if you use `at` instead of `within`, the resulting bindings are *anchored*.  Anchored bindings only override direct dependencies of the context they're applied to, whereas unanchored ones (produced by `within`) override bindings for transitive dependencies as well.

### Parameters

[NeighborhoodSize]: http://lenskit.grouplens.org/apidocs/org/grouplens/lenskit/knn/params/NeighborhoodSize.html

LensKit provides many parameters, which are annotated with various
annotations (such as [NeighborhoodSize][].  These parameters are set
using the `set` method:

~~~java
factory.set(NeighborhoodSize.class).to(50);
~~~

Type safety is somewhat relaxed for parameters, but they are used for
numeric or occasionally string values.

### Qualifiers 

Parameters are a special case of the more general concept of
''qualifiers'' — annotations which are annotated with `@Qualifier`
from JSR 330, and are used to specify additional distinctions between
objects. You can bind one using the two-parameter version of `bind`:

~~~java
factory.bind(Qualifier.class, ComponentType.class)
       .to(ComponentImpl.class);
~~~

Alternatively, you can use the `withQualifier` method of `Binding`:

~~~java
factory.bind(ComponentType.class)
       .withQualifier(Qualifier.class)
       .to(ComponentImpl.class);
~~~

Qualifiers are used in a couple of places:

- To specify parameters, such as damping terms, that have primitive or
  string values.
- To distinguish when a class depends on multiple components of the
  same type.

Other DI frameworks, such as Guice, encourage much broader use of qualifiers than we use in LensKit. Contexts provide a preferable solution in many cases.

### Groovy

LensKit supports a Groovy-based DSL for configuration as well.  This is used in the [evaluator](../evaluator/), and can be used by applications via the `lenskit-groovy` module.  Here is an example of a configuration in the DSL:

~~~groovy
bind ItemScorer to ItemItemScorer
within(UserVectorNormalizer) {
  bind VectorNormalizer to MeanVarianceNormalizer
}
~~~

[load]: http://lenskit.org/apidocs/org/grouplens/lenskit/config/ConfigHelpers.html#load%28java.io.File%29
[ConfigurationLoader]: http://lenskit.org/apidocs/org/grouplens/lenskit/config/ConfigurationLoader.html

You can load this by using [ConfigHelpers.load][load] or the [ConfigurationLoader][] class.
