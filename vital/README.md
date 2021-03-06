Here lies the vital library, the core components of our framework from
exceptions, to common type definitions and utilities, to configuration and the
plugin management.
Oh, a bunch of plugin interfaces.

# Sub-Directory Layout
The layout here likely needs some refactoring.
E.g. one complaint has been some directories become overly populated, so it is
hard to find any particular thing, i.e. the `algo` and `types` subdirectories.

Option: move everything required for configuration, logging and plugin
management into a "core" subdirectory and library. Move out the logger, config
and applet plugins that are included here for separate module building in a
different subdirectory tree that is to be descended *after* building core +
other vital things (like interfaces and types, etc.).


# Things specifically changed/removed from KWIVER 1.x
* Removed `vital/any.h` with the updated base CXX standard --> **17**.


# Design

## Plugin management modifications
### Plugin Factory
Added a `pluggable` type to act as a base class handle for pluggable things so
that the factory could define a pure-virtual for instance creation and return a
shared-ptr.

Changed the instance creation method to take in a config block instead of
being empty.
The concrete, templated implementation class of the base factory interface,
concrete_plugin_factory, is header only and requires:
* the concrete class inherit from the given interface class
* that the concrete have the static methods `from_config` and
  `get_default_config`. These are documented in the `plugin_factory`
  -equivalent methods.

Standardizing on:
* interface name taken from `T::interface_name()` static method.
* concrete name taken from `typeid().name()` result.

### Plugin Loader
Removed plugin filters. Linus described these as deprecated in lieu of the use
of plugin types at the manager level.

"Deprecated" a number of methods on the loader that don't seem to be used in
ernest.
I may be wrong about these and just have only commented them out for now.

### Plugin Manager
Similarly, deprecated a number of functions as in the loader.
Same comment about my potential wrongness.

Did not carry over yet the "internal" sub-class just yet, also pending exposure
of a reason for existence.

## Plugin How-To
### Loading Plugins
Register plugins by running
```c++
kwiver::vital::plugin_manager::instance()->load_all_plugins();
```
Additional search paths may be added for discovery.
This may be done programatically via the `plugin_manager` instance or via
the environment variable `KWIVER_PLUGIN_PATH`.
The environment viariable should be filesystem paths to directories, separated
by your system's usual PATH-separator.

### Registering Plugins
In your CMake, use `kwiver_add_plugin` to create shared plugin libraries.
This library should be given at least a source file that defines an extern-C
function `extern "C" void register_factories( kwiver::vital::plugin_loader& vpl )`
that adds implementation factories to the given loader instance.
The header ``<vital/plugin_management/plugin_loader.h>`` should be included to
support implementing the above function.
This function signature is constant and is defined by the vital plugin
manager/loader.

It is _**important**_ that this function be attributed with the appropriate
export method, otherwise this function symbol will not be visible when
dynamically loading the plugin library.
When using the KWIVER cmake utility `kwiver_add_plugin`, a `<NAME>_export.h`
header file will automatically be generated in the build tree, where `<NAME>` is
the plugin library name provided to the CMake function.
For example, if calling `kwiver_add_plugin( foo ... )`, then the header file
`foo_export.h` will be generated.
The `register_factories` function implementation signature might then look like
```c++
#include <.../foo_export.h>

extern "C"
FOO_EXPORT
void
register_factories( kwiver::vital::plugin_loader& vpl )
{
  // ...
}
```

When implementing the above function, the most common method of registering C++
concrete types is via the method:
```c++
vpl->add_factory<INTERFACE, CONCRETE>( "my-concrete-plugin-name" );
```
Where `INTERFACE` is the type of the interface being implemented and `CONCRETE`
is the concrete implementation type implementing that interface.
This may raise compilation errors if the interface and concrete types do not
implement the appropriate static methods for plugin support, documented in
`vital/plugin_management/pluggable.h`.
