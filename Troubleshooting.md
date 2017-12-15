To trigger a **complete rebuild**, run `make clean`. This will remove all local build artifacts. for all platforms. To selectively delete just the build artifacts for a particular platform, delete the respective folder in the `build` directory.

If you have **[[installed ccache|Workflow → ccache]]**, and if you suspect that it cached a bad artifact, run `ccache --cleanup` to clear the cache. 

If you are having trouble getting the **dependencies** right, you can delete the `mason_packages` directory, or run `make distclean`. This means the Makefile and [[CMake|Workflow → CMake]] will automatically install the dependencies again on the next try.
