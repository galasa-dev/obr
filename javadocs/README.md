# How does the Javadoc get built ?

> Look at the build-locally to see what the sequence of steps is to build it.

> Note: These notes would have been added to the `pom.xml`, but as that is generated from the `pom.template`, and the template isn't allowed to contain comments (!), a separate `README` seemed appropriate.

But basically it does this:

1. The galasabld tool is used to take the `pom.template` and all the various `release.yaml` files in sibling projects, and constructs a generated `pom.xml`

2. The maven build using that generated pom is executed.

   - All the dependencies marked as 'sources' are pulled down and expanded (into `target/sources`)
     - This means that all the source code is landed in one giant source tree and processed together, with all the cross-linkages being resolvable. 
     - Mosty the content comes from `managers`, `framework` and `extensions`.
   - All the dependencies which are java classes are pulled down and expanded (into `target/classes`)
     - That's a ton of classes. May take a while to download locally the first time around.
   - Javadoc gets called 
     - All those classes are on the classpath, so that every reference to anything can be resolved.
     - We use the `--frames` flag, causing iFrames to appear in the html output. This option is deprecated, and will be removed in future releases of javadoc. But not now, so we are continuing to use it.
   - The output from javadoc is assembled into a zip, and installed into the local maven repo.

3. The main build process then takes the zip and puts it into a maven repository structure, and packages it as a maven repo. The image is pushed to Harbor.
4. The maven repo container image is deployed to the external site.
5. The unpacked site html files are packaged into a docker image.
6. The 'site' docker image is not deployed until we do a release.


