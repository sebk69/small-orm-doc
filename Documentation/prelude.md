# small-orm documentation

[back to table of content](table-of-content.md)

## Prelude

small-orm was originally developed for Symphony and some Symfony terms like 'bundles' come form that reason.

In small-orm vocabulary, a bundle is a set of DAO/Model with some isolations (but your can add relations between bundles).

Originally it was developped to package some generics bundles like user management which could be imported in many bundles.

Often, in my projects, I use bundles to structure app in functionnal sections (ex: customers models, order models...).

The Swoft implementation keep this structure, even if bundles are not really submodules.