## Modules

Modules are how we organize segments of code into logical containers, like
a math library for instance. We can seperate modules across files and import
them. Again Melodeon's module system has been inspired by Rust, so the Rust
docs are a great resource to understand modules and project structure.

### Project Structure

We've seen how we can run an individual file with `melorun example.melo`. But
we can also provide a directory to melorun, `melorun ./example-project/`.
Melorun will then interpret this directory as the root of a well structured
Melodeon project. **The primary rule for a project is a main.melo file at the
root directory.**

Modules can be imported at the top of a file, using paths starting from the
root directory.

```
use signed_int;
```

This imports the signed_int module which is searched for by melorun from the
root directory. There are two ways melorun can find this module.

- There is a signed_int.melo file in the root directory
- There is a signed_int sub-directory in the root, with a main.melo file
  inside. This main.melo file is the entry-point of the signed_int module.

Modules can also be nested in sub-directories, and importing would look like:

```
use math/signed_int;
```

Note this is a Melodeon-specific path representation, not an OS path string.
The `/` used is not specific to any OS, it is just used to denote
sub-structures.

### Exposing Definitions from a module

TODO: ideally this should be a little less verbose, like just adding "pub" to
a definition.

By default all definitions in a module are private (not accessible outside the
module). In order to make a definition accessible, add a provide clause to the
top of the file.

```
provide foo;

def foo() = "something useful"
```
