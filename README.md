# Yireo ExtensionChecker
This extension validates the code of other extensions and is complementary to static code analysis tools like PHPCS.

## Example 1: Scanning extension code

    bin/magento yireo_extensionchecker:scan Yireo_ExampleAdminhtml

Running this command might give the following output:

    Dependency "Magento_Backend" not found module.xml
    Dependency "Magento_Ui" not found module.xml
    Dependency "magento/module-backend" not found composer.json. Current version is 101.0.2
    Dependency "magento/module-ui" not found composer.json. Current version is 101.1.2
    Dependency "psr/log" not found composer.json. Current version is 1.1.0

The output gives a hint to what to add to `composer.json`. For instance, a composer requirement `magento/module-ui` should be added and this could have a version constraint `^101.1` to match semantic versioning. Theoretically, this could also be `^101.0` or even `^100.0|^101.0`, but for this, deep-code analysis (by you) would be needed.

## Example 2: Comparing composer versions

    bin/magento yireo_extensionchecker:version_match /path/to/your/extension/composer.json

This command allows you to scan a given `composer.json` of some extension, without that extension being installed. This allows you to modify the dependencies of that extension, before trying to install the extension (which could save valuable time).

The output should be empty to be successful. When unmatched versions are found, it might look like the following:  

    ERROR: "magento/framework:103.0.0" does not match required version "^100.1|^101.0|^102.0"

## Example 3: Creating unit tests (@todo: Move this to another module
```bash
bin/magento yireo_extensionchecker:generate-unit-test --module Yireo_Webp2 --class '\Yireo\Webp2\Convertor\Convertor'
```

## Installation
Install the module as a composer requirement for developer environments:

    composer require --dev yireo/magento2-extensionchecker
    bin/magento module:enable Yireo_ExtensionChecker
    
Note that if you want to scan a module, this module also needs to be enabled. Personally, we use this extension in our CI/CD chain, to make sure zero issues are reported at all times.

## Deprecated dependencies
Class dependencies (injected via the constructor) are inspected to see if they are deprecated, for the used Magento version. You can skip this behaviour by adding a flag `--hide-deprecated` to the command:

    bin/magento yireo_extensionchecker:scan Yireo_Example --hide-deprecated=1

## Undeclared dependencies
Class dependencies (injected via the constructor) are traced back to their corresponding module (or the framework or something else), which should be reflected upon in the `composer.json` file and the `module.xml` file. Of each composer dependencies, the current version is also reported.

Also, by tokenizing the PHP source, it is detected whether the `composer.json` file should reflect a specific PHP extension (for example, `ext-json`) when an extension-specific PHP function is used (for example, `json_encode`).

## @todo: Incorrect versioning for dependencies
When loading dependencies in `composer.json`, semantic versioning should be used to identify the right release for your dependency (major, minor, patch). A wildcard `*` is definitely forbidden. Magento dependencies should be in proper format. All dependencies should have major definitions that are not in the future.

## @todo: Hard-coded Proxies
A Proxy is a DI trick which should be configured in the `di.xml` file of a module and not be hard-coded in PHP. The extension could report this.

## @todo: Check other methods for signature
If another method than the constructor contains type hints for imported namespaces, those namespaces lead to further dependencies with the module. For example, if a specific method returns an object of type `Magento/ModuleX/SomeInterface` then `Magento_ModuleX` would need to be reported as a dependency.

## @todo: Check for XML files
Some XML files lead to the requirement (or suggestion?) for other modules. For instance, the existence of a folder `etc/adminhtml/` would require the `Magento_Adminhtml` module.

## @todo: Scan for `@since`
Scan class dependencies for `@since` and double-check if this minimum version matches with the composer requirements.

## Tip: Check multiple modules 
You can quickly check upon multiple modules with a command like this:

    bin/magento mod:st --enabled | grep Yireo_ | while read MODULE ; do 
        echo "Checking $MODULE"
        bin/magento yireo_extensionchecker:scan $MODULE --hide-deprecated 1
    done
