When working on a change that depends on a change to mapbox-gl-js (shaders, integration tests, style spec), use the followin workflow:

1. Open a work-in-progress PR/branch in `mapbox-gl-js` with the changes needed there.  (Example: https://github.com/mapbox/mapbox-gl-js/pull/4212 un-skips the integration tests relevant to enabling DDS for text-field, text-transform.)
2. Open a PR with the proposed changes here, including a commit that updates the mapbox-gl-js submodule to the wip branch from step 1) ☝️. 
3. While waiting for or responding to reviews, `mapbox-gl-native/master` and `mapbox-gl-js/master` may continue to move along, including the possibility that `-native/master` needs to track updates in `gl-js/master`.  So, from time to time, rebase the branch in step 1) and then the PR branch here.
4. Once we're 🍏, merge the PR to mapbox-gl-js. If it can be fast-forward merged, then do that (via the command line) -- this saves having to reupdate the submodule SHA in mapbox-gl-native. Otherwise, rebase and merge. Then update the submodule SHA in mapbox-gl-native if necessary, and merge the mapbox-gl-native PR.
